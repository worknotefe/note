### node-glob 的匹配规则
- * 匹配任意 0 或多个任意字符
- ? 匹配任意一个字符
- [...] 若字符在中括号中，则匹配。若以 ! 或 ^ 开头，若字符不在中括号中，则匹配
- !(pattern|pattern|pattern) 不满足括号中的所有模式则匹配
- ?(pattern|pattern|pattern) 满足 0 或 1 括号中的模式则匹配
- +(pattern|pattern|pattern) 满足 1 或 更多括号中的模式则匹配
- *(a|b|c) 满足 0 或 更多括号中的模式则匹配
- @(pattern|pat*|pat?erN) 满足 1 个括号中的模式则匹配
- ** 跨路径匹配任意字符
