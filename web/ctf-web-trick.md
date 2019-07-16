# CTF-Web Trick

1. 拿到shell后找不到flag：

   ```bash
   find / | grep flag 2>/dev/null
   ```

   还可以检查环境变量、host、http log来找flag

2. 管理员线上改题目，赶紧找有没有 .index.swp .index.swo

