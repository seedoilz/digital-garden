---
aliases: 
title: Picocli
date created: 二月 25日 2024, 9:19:01 晚上
date modified: 三月 5日 2024, 4:07:11 下午
tags: [code/tools, language/java]
---

> [!NOTE] Picocli是什么
> 命令行工具开发框架，支持颜色高亮、自动补全、子命令、帮助手册……

### 帮助手册（--help）
>通过给类添加 `@Command` 注解参数mixinStandardHelpOptions设置true来开启

```java
@Command(name = "ASCIIArt", mixinStandardHelpOptions = true)
```

### 命令解析

Picocli 最核心的能力就是**命令解析**，能够从一句完整的命令中解析选项和参数，并填充到对象的属性中
使用 `@Option` 和 `@Parameters` 注解用于解析参数



> [!NOTE] 注意⚠️
> 如果execute中的参数中没有"-cp"，则不会进行输入。

```java
public static void main(String[] args) {
    new CommandLine(new Login()).execute("-u", "user123", "-p", //"-cp");
}
```

### 交互式输入

`@Option` 注解的 interacitvie设置为true，表示该项支持交互式输入
```java
@Option(names = {"-p", "--password"}, description = "Passphrase", interactive = true)
```

#### 可选交互式

##### 问题
>如果命令中包含-p xxx会**报错**，原因：参数不匹配
```java
public static void main(String[] args) {
    new CommandLine(new Login()).execute(
							    "-u", "user123", "-p", "xxx", "-cp");
}
```
##### 解决方案
添加 `arity = "0.. 1"` 来指定每个选项可接收的参数个数
```java
@Option(names = {"-p", "--password"}, arity = "0..1", description = "Passphrase", interactive = true)
String password;
```

#### 强制交互式
>编写一段通用的校验程序，如果用户的输入命令中没有包含交互式选项，那么就自动为输入命令补充该选项，强制触发交互式输入

```java

public static String[] processInteractiveOptions(Class<?> clazz, String[] args) {
	// 将传递过来的数组转成集合，方便添加
	Set<String> argSet = new LinkedHashSet<>(Arrays.asList(args));

	// 获取字段的Option注解
	for (Field field : clazz.getDeclaredFields()) {
		// 如果注解存在且其interactive属性为true，则执行以下操作
		Option option = field.getAnnotation(Option.class);
		if (option != null && option.interactive()) {
			// 如果传递的参数中没有该属性，则添加
			if (!argSet.contains(option.names()[0])) {
				argSet.add(option.names()[0]);
			}
		}
	}
	args = argSet.toArray(new String[0]);
	return args;
}
```

### 子命令

#### 声明式
通过@Command注解的subcommands属性来给命令添加子命令，优点是直观清晰
```java
@Command(subcommands = {
    GitCommit.class,
    GitAdd.class,
    GitBranch.class,
    GitCheckout.class,
    GitClone.class,
    GitPush.class
})
public class Git { /* ... */ }
```

#### 编程式
```java
CommandLine commandLine = new CommandLine(new Git())
        .addSubcommand("commit",   new GitCommit())
        .addSubcommand("add",      new GitAdd())
        .addSubcommand("branch",   new GitBranch())
        .addSubcommand("checkout", new GitCheckout())
        .addSubcommand("clone",    new GitClone())
        .addSubcommand("push",     new GitPush())
        )
```

#### 实践
```java
package com.seedoilz.cli.example;

import picocli.CommandLine;
import picocli.CommandLine.Command;

@Command(name = "main", mixinStandardHelpOptions = true)
public class SubCommandExample implements Runnable {

    @Override
    public void run() {
        System.out.println("执行主命令");
    }

    @Command(name = "add", description = "增加", mixinStandardHelpOptions = true)
    static class AddCommand implements Runnable {
        public void run() {
            System.out.println("执行增加命令");
        }
    }

    @Command(name = "delete", description = "删除", mixinStandardHelpOptions = true)
    static class DeleteCommand implements Runnable {
        public void run() {
            System.out.println("执行删除命令");
        }
    }

    @Command(name = "query", description = "查询", mixinStandardHelpOptions = true)
    static class QueryCommand implements Runnable {
        public void run() {
            System.out.println("执行查询命令");
        }
    }

    public static void main(String[] args) {
        // 执行主命令
        String[] myArgs = new String[]{};
        // 查看主命令的帮助手册
//        String[] myArgs = new String[]{"--help"};
        // 执行增加命令
//        String[] myArgs = new String[]{"add"};
        // 执行删除命令的帮助手册
//        String[] myArgs = new String[]{"delete", "--help"};
        // 执行不存在的命令，会报错
//        String[] myArgs = new String[]{"update"};

        int exitCode = new CommandLine(new SubCommandExample())
                .addSubcommand(new AddCommand())
                .addSubcommand(new DeleteCommand())
                .addSubcommand(new QueryCommand())
                .execute(myArgs);
        System.exit(exitCode);
    }
}
```