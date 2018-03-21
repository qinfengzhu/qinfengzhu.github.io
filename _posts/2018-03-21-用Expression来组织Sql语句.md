---
layout: post
title: "用Expression来组织Sql语句"
date: 2018-03-21   
tag: 工具 
---

### 具体思路

### 实现的关键点

```csharp
/// <summary>获取属性名称</summary>
/// <param name="expression">属性表达式</param>
/// <returns>属性名称</returns>
public static string GetPropertyName(Expression expression)
{
	var body = expression;
	var le = body as LambdaExpression;
	if(le != null)
		body = le.Body;
	var ue = body as UnaryExpression;
	if(ue != null)
		body = ue.Operand;
	var me = body as MemberExpression;
	if(me != null)
		return me.Member.Name;

	throw new InvalidOperationException();
}
```


```csharp
/// <summary>
/// 获取二元表达式右则的值
/// </summary>
/// <param name="expression">表达式</param>
/// <returns>二元表达式右则计算出来的值</returns>
protected virtual object GetRightValue(Expression expression)
{
	if(expression.NodeType == ExpressionType.Constant)
		return ((ConstantExpression)expression).Value;
	return Expression.Lambda<Func<object>>(Expression.Convert(expression, typeof(object)), new ParameterExpression[0]).Compile()();
}
```