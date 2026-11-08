# 使用服务器会话认证浏览器应用

状态：accepted，依据用户对 Q12 的确认。

平台只有自有浏览器 SPA，开发通过代理、部署通过 Nginx 同源访问，采用 Spring Security 服务器 Session、HttpOnly Cookie 与 CSRF 防护，替代旧规划尚未确认的 JWT 建议。这样退出、停用与授权变化可以通过会话失效或重新鉴权处理，避免第一版引入 access/refresh token 轮换。代价是首版单实例内存会话不持久化，重启须重新登录；将来多实例部署需重新选择共享会话存储并验证相关配置。
