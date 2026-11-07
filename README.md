# 基于 CentOS 8 的 Nginx + MariaDB 动态网站部署与自动化运维

## 📌 项目概述
本项目在 VMware 虚拟机中基于 CentOS 8 操作系统，独立完成了 Nginx Web 服务器与 MariaDB 数据库的部署。通过配置 PHP-FPM 实现 Nginx 解析动态 PHP 页面，搭建了一个具备完整“增删查”功能的动态留言板网站。此外，结合 Shell 脚本与 crontab 实现了数据库的每日自动备份，完整实践了从系统初始化、服务搭建、应用联调到自动化运维的全流程。

![留言板最终效果]（https://github.com/buster34101/centos-nginx-mariadb-deploy/blob/main/images/images/04-guestbook.png）
## 🏗️ 环境与架构
- **操作系统**：CentOS 8（VMware 虚拟机，NAT 网络模式）
- **Web 服务**：Nginx 1.14
- **数据库**：MariaDB 10.3
- **运行环境**：PHP 7.2 (php-fpm)
- **架构说明**：客户端发起请求 -> Nginx 处理静态资源及反向代理 -> 遇到 `.php` 文件转发给 PHP-FPM 解析 -> PHP 通过 `mysqli` 扩展读写 MariaDB 数据库。

## 🛠️ 实施步骤
1. **系统环境初始化**：配置阿里云 vault 源解决 CentOS 8 EOL 问题；临时关闭 SELinux 防止 Nginx 访问受限；配置 `firewalld` 放行 HTTP(80) 服务。
2. **Web 环境搭建**：通过 `yum` 安装 Nginx，替换默认静态页面，配置 Nginx 配置文件实现 `fastcgi_pass` 转发。
3. **数据库部署**：安装并启动 MariaDB，设置 root 密码，创建 `webapp` 数据库及 `messages` 数据表，完成基础 SQL 增删改查验证。
4. **前后端联调**：编写 `index.php` 页面，实现 HTML 表单提交与 PHP 接收数据，通过 `mysqli_connect` 连接数据库，实现留言的写入与页面列表动态渲染。
5. **自动化运维**：编写 `backup.sh` 脚本，使用 `mysqldump` 导出数据库，配置 `crontab` 定时任务实现每天凌晨 2 点自动备份。

## 🚧 遇到的问题与解决（面试重点）
**1. 宿主机无法通过浏览器访问虚拟机 Nginx 页面**
- **排查过程**：检查虚拟机 `ss -tlnp | grep :80` 确认 Nginx 监听 `0.0.0.0:80`；检查 `firewall-cmd --list-all` 确认 HTTP 服务已放行；在宿主机使用 `curl` 发现能正常获取页面源码。
- **定位原因**：浏览器默认开启了代理服务，导致无法访问内网 NAT 地址。
- **解决**：配置浏览器代理绕过本地 IP，成功访问。

**2. Nginx 启动报错：unknown directive**
- **排查过程**：修改 `nginx.conf` 添加 PHP 解析配置后，运行 `systemctl restart nginx` 失败。
- **定位原因**：`echo` 命令写入脚本时，`$document_root` 等变量被 Shell 当作环境变量解析为空，导致 Nginx 配置文件语法残缺；且 `location` 代码块嵌套错误。
- **解决**：使用 `vi` 编辑器手动重写 `default.d/php.conf` 文件，通过 `nginx -t` 测试通过后重启，问题解决。

## 📈 收获与改进
- **收获**：熟悉了 Linux 系统下的包管理、服务控制与防火墙配置；掌握了 Nginx 与 PHP-FPM 的协同工作机制；复习了 SQL 语法与 PHP 基础；理解了自动化备份脚本在实际运维中的价值。
- **后续改进方向**：后续可引入 Docker 容器化部署该架构；为 Nginx 配置 HTTPS 证书（SSL）；使用 Nginx 实现多站点反向代理。
