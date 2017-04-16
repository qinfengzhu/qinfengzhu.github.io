---
layout: post
title: RegularExpression基本应用
date: 2017-04-15
tags: 工具    
---

### 正则表达式如此重要

	NPoco用正则来对Sql语句分解验证,AutoMapper用来处理名称,Newtonsoft用来校验所得字符串是否满足某种值类型等等。其实就是主要做一些字符检验提取的工作，效率奇高。

	后面的测试都会用到一个[在线正则表达式工具](http://www.regexr.com/)
	
>* NPoco 的实例

```csharp

	 	public static Regex rxColumns = new Regex(@"\A\s*SELECT\s+((?:\((?>\((?<depth>)|\)(?<-depth>)|.?)*(?(depth)(?!))\)|.)*?)(?<!,\s+)\bFROM\b", RegexOptions.IgnoreCase | RegexOptions.Multiline | RegexOptions.Singleline | RegexOptions.Compiled);
        public static Regex rxOrderBy = new Regex(@"\bORDER\s+BY\s+(?:\((?>\((?<depth>)|\)(?<-depth>)|.?)*(?(depth)(?!))\)|[\w\(\)\.\[\]""`])+(?:\s+(?:ASC|DESC))?(?:\s*,\s*(?:\((?>\((?<depth>)|\)(?<-depth>)|.?)*(?(depth)(?!))\)|[\w\(\)\.\[\]""`])+(?:\s+(?:ASC|DESC))?)*", RegexOptions.IgnoreCase | RegexOptions.Multiline | RegexOptions.Singleline | RegexOptions.Compiled);
        public static Regex rxDistinct = new Regex(@"\ADISTINCT\s", RegexOptions.IgnoreCase | RegexOptions.Multiline | RegexOptions.Singleline | RegexOptions.Compiled);
    
```	

>* AutoMapper 的实例

```csharp

	private readonly Regex _splittingExpression = new Regex(@"(\p{Lu}+(?=$|\p{Lu}[\p{Ll}0-9])|\p{Lu}?[\p{Ll}0-9]+)");
	private readonly Regex _splittingExpression = new Regex(@"[\p{Ll}0-9]+(?=_?)");

```

>* Newtosoft 的实例

```csharp

	public const string EmailAddressRegex = @"^([a-zA-Z0-9_'+*$%\^&!\.\-])+\@(([a-zA-Z0-9\-])+\.)+([a-zA-Z0-9:]{2,4})+$";
    public const string CurrencyRegex = @"(^\$?(?!0,?\d)\d{1,3}(,?\d{3})*(\.\d\d)?)$";
    public const string DateRegex =
        @"^(((0?[1-9]|[12]\d|3[01])[\.\-\/](0?[13578]|1[02])[\.\-\/]((1[6-9]|[2-9]\d)?\d{2}|\d))|((0?[1-9]|[12]\d|30)[\.\-\/](0?[13456789]|1[012])[\.\-\/]((1[6-9]|[2-9]\d)?\d{2}|\d))|((0?[1-9]|1\d|2[0-8])[\.\-\/]0?2[\.\-\/]((1[6-9]|[2-9]\d)?\d{2}|\d))|(29[\.\-\/]0?2[\.\-\/]((1[6-9]|[2-9]\d)?(0[48]|[2468][048]|[13579][26])|((16|[2468][048]|[3579][26])00)|00|[048])))$";
    public const string NumericRegex = @"\d*";

```

### 正则表达式学习步骤

