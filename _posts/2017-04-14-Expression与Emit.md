---
layout: post
title: Expression与Emit
date: 2017-04-14
tags: 工具    
---


### Expression与Emit做什么
	
	Expression动态的生成方法,或者根据已有的Expression获取想要的数据。而Emit是出现在Expression之前,Emit的出现主要是为了解决System.Reflection中提供反射处理方式性能不佳的方案。

	日常中我们一般用Expression 与 Emit做什么好呢，延缓后续DynamicMethod处理基础类赋值功能。


### 边看代码边介绍,这里以AutoMapper代码为例

```csharp
namespace AutoMapper
{

	internal delegate object LateBoundMethod(object target, object[] arguments);//方法调用委托
	internal delegate object LateBoundPropertyGet(object target);//获取属性值委托
	internal delegate object LateBoundFieldGet(object target);//获取字段值委托
	internal delegate void LateBoundFieldSet(object target, object value);//设置字段字委托
	internal delegate void LateBoundPropertySet(object target, object value);//设置属性值委托
	internal delegate void LateBoundValueTypeFieldSet(ref object target, object value);//设置引用值类型值委托
	internal delegate void LateBoundValueTypePropertySet(ref object target, object value);//设置引用属性类型值的委托
	internal delegate object LateBoundCtor();//构造函数委托

    //委托工厂
	internal static class DelegateFactory
	{
        private static readonly object _ctorCacheSync = new object();//类型构造锁，主要是为操作Dictionary<Type,LateBoundCtor> 解决并发问题
		private static readonly Dictionary<Type, LateBoundCtor> _ctorCache = new Dictionary<Type, LateBoundCtor>();//类型与构造方法委托
		
		//Expression 构建方法委托，参数为MethodInfo: 提供DeclaringType 与 Parameters 信息
		public static LateBoundMethod CreateGet(MethodInfo method)
		{
			ParameterExpression instanceParameter = Expression.Parameter(typeof(object), "target");
			ParameterExpression argumentsParameter = Expression.Parameter(typeof(object[]), "arguments");

			MethodCallExpression call = Expression.Call(
				Expression.Convert(instanceParameter, method.DeclaringType),
				method,
				CreateParameterExpressions(method, argumentsParameter));

			Expression<LateBoundMethod> lambda = Expression.Lambda<LateBoundMethod>(
				Expression.Convert(call, typeof(object)),
				instanceParameter,
				argumentsParameter);

			return lambda.Compile();
		}

	    //Expression 构建获取属性值的委托,参数PropertyInfo：提供DeclaringType 信息
		public static LateBoundPropertyGet CreateGet(PropertyInfo property)
		{
			ParameterExpression instanceParameter = Expression.Parameter(typeof(object), "target");

			MemberExpression member = Expression.Property(Expression.Convert(instanceParameter, property.DeclaringType), property);

			Expression<LateBoundPropertyGet> lambda = Expression.Lambda<LateBoundPropertyGet>(
				Expression.Convert(member, typeof(object)),
				instanceParameter
				);

			return lambda.Compile();
		}
		//Expression 构建获取字段值的委托,参数FieldInfo: 提供DeclaringType 信息
		public static LateBoundFieldGet CreateGet(FieldInfo field)
		{
			ParameterExpression instanceParameter = Expression.Parameter(typeof(object), "target");

			MemberExpression member = Expression.Field(Expression.Convert(instanceParameter, field.DeclaringType), field);

			Expression<LateBoundFieldGet> lambda = Expression.Lambda<LateBoundFieldGet>(
				Expression.Convert(member, typeof(object)),
				instanceParameter
				);

			return lambda.Compile();
		}
		//Emit 构建设置字段值的委托,参数FieldInfo：提供DeclaringType 与 FieldType 信息
		public static LateBoundFieldSet CreateSet(FieldInfo field)
		{
			var sourceType = field.DeclaringType;

            var method = CreateDynamicMethod(field, sourceType);

            var gen = method.GetILGenerator();

			gen.Emit(OpCodes.Ldarg_0); // Load input to stack
			gen.Emit(OpCodes.Castclass, sourceType); // Cast to source type
			gen.Emit(OpCodes.Ldarg_1); // Load value to stack
			gen.Emit(OpCodes.Unbox_Any, field.FieldType); // Unbox the value to its proper value type
			gen.Emit(OpCodes.Stfld, field); // Set the value to the input field
			gen.Emit(OpCodes.Ret);

			var callback = (LateBoundFieldSet)method.CreateDelegate(typeof(LateBoundFieldSet));

			return callback;
		}
		//Emit 构建设置字段值的委托,参数PropertyInfo：提供DeclaringType、SetMethod 与 FieldType 信息
	    public static LateBoundPropertySet CreateSet(PropertyInfo property)
		{
			var sourceType = property.DeclaringType;
			var setter = property.GetSetMethod(true);
			var method = CreateDynamicMethod(property, sourceType);
            var gen = method.GetILGenerator();

			gen.Emit(OpCodes.Ldarg_0); // Load input to stack
			gen.Emit(OpCodes.Castclass, sourceType); // Cast to source type
			gen.Emit(OpCodes.Ldarg_1); // Load value to stack
			gen.Emit(OpCodes.Unbox_Any, property.PropertyType); // Unbox the value to its proper value type
			gen.Emit(OpCodes.Callvirt, setter); // Call the setter method
			gen.Emit(OpCodes.Ret);

			var result = (LateBoundPropertySet)method.CreateDelegate(typeof(LateBoundPropertySet));

			return result;
		}

	    public static LateBoundValueTypePropertySet CreateValueTypeSet(PropertyInfo property)
		{
			var sourceType = property.DeclaringType;
			var setter = property.GetSetMethod(true);
			var method = CreateValueTypeDynamicMethod(property, sourceType);
			var gen = method.GetILGenerator();

			method.InitLocals = true;
			gen.Emit(OpCodes.Ldarg_0); // Load input to stack
			gen.Emit(OpCodes.Ldind_Ref);
			gen.Emit(OpCodes.Unbox_Any, sourceType); // Unbox the source to its correct type
			gen.Emit(OpCodes.Stloc_0); // Store the unboxed input on the stack
			gen.Emit(OpCodes.Ldloca_S, 0);
			gen.Emit(OpCodes.Ldarg_1); // Load value to stack
			gen.Emit(OpCodes.Castclass, property.PropertyType); // Unbox the value to its proper value type
			gen.Emit(OpCodes.Call, setter); // Call the setter method
			gen.Emit(OpCodes.Ret);

			var result = (LateBoundValueTypePropertySet)method.CreateDelegate(typeof(LateBoundValueTypePropertySet));

			return result;
		}
		//构建类型与委托的缓存
	    public static LateBoundCtor CreateCtor(Type type)
		{
			LateBoundCtor ctor;
			if (!_ctorCache.TryGetValue(type, out ctor))
			{
                lock (_ctorCacheSync)
				{
                    if (_ctorCache.TryGetValue(type, out ctor))
					{
						return ctor;
					}

					var ctorExpression = Expression.Lambda<LateBoundCtor>(Expression.Convert(Expression.New(type), typeof(object)));
					ctor = ctorExpression.Compile();
					_ctorCache.Add(type, ctor);
				}
			}
			return ctor;
		}
		
		private static DynamicMethod CreateValueTypeDynamicMethod(MemberInfo member, Type sourceType)
	    {
	    	if (sourceType.IsInterface)
                return new DynamicMethod("Set" + member.Name, null, new[] { typeof(object).MakeByRefType(), typeof(object) }, sourceType.Assembly.ManifestModule, true);

	        return new DynamicMethod("Set" + member.Name, null, new[] { typeof(object).MakeByRefType(), typeof(object) }, sourceType, true);
	    }
	    private static DynamicMethod CreateDynamicMethod(MemberInfo member, Type sourceType)
	    {
	    	if (sourceType.IsInterface)
	            return new DynamicMethod("Set" + member.Name, null, new[] { typeof(object), typeof(object) }, sourceType.Assembly.ManifestModule, true);

	        return new DynamicMethod("Set" + member.Name, null, new[] { typeof(object), typeof(object) }, sourceType, true);
	    }
	    private static Expression[] CreateParameterExpressions(MethodInfo method, Expression argumentsParameter)
		{
			return method.GetParameters().Select((parameter, index) =>
				Expression.Convert(
					Expression.ArrayIndex(argumentsParameter, Expression.Constant(index)),
					parameter.ParameterType)).ToArray();
		}
}
```

关于Expression的具体使用，见[Expression详细使用讲解](http://blog.csdn.net/zhuqinfeng/article/details/70168337).