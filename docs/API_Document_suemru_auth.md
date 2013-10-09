#API 文档

sumeru.auth客户端API

##sumeu.auth.on(type,handle)

增加一个用户系统相关的事件监听器. 

**type** : string : 目前只支持一个事件类型,即 **“statusChange”**, 每次认证状态变化时被触发.  
**handle** : function : 事件处理函数. 可接收两个参数err与status, err为有错误产生时的错误对像, status 为当前认证状态的字符串表示. 

> statusChange事件被触发时，传入status的值可能为以下以种：
>
> + “not_login”	当前未登陆
> + “logined”  已登陆
> + “doing_login” 正在登陆过程中

EXAMPLE CODE:

		sumeru.auth.on('statusChange',function(err,status){  
			if(err){
				// err.code | err.msg
				return;
			};  
		  
			switch(status){  
				case "not_login" :
					// do something
					break;
				case "logined" :
					// do something
					break;
				case "doing_login" :
					// do something
					break;
				default:
					// do something
			}
		});



> _每个监听器在被添加时，都将自动被调用一次，用来通知客户端当前的认证状态。_

##Sumeru.auth.removeAllListener(type)
##Sumeru.auth.removeListener(type,handle)

移除由on方法增加的监听器 , 其中removeAllListener一次装移除所有已添加的监听器事件。

**type** : string : 事件名称, 目前只支持一个事件,即 **“statusChange”**  
**handle** : function 事件处理函数

##sumeru.auth.login(token,pwd,[args],[authMethod]);

根据token,pwd登陆由type所指定的类型用户系统. 不提供type时默认为local.
登陆过程中的每次状态变化将触发statusChange事件

**token** : string : 用户名  
**pwd** : string : 密码  
**args** : map : 登陆时需附加的其它信息,具体内容根据type的不同传入内容将不同.如不提供默认为{}, **所有除用户名和密码外需要传入的其它数据（如验证码等），都需要通过这个map对像传入，每个authMethod所要求的参数可能并不相同（取决于对应authMethod的相关实现，参见：*第三方认证扩展* ）  **  
**authMethod** : string : 登陆用户的类型. 默认为’local’

EXAMPLE CODE:

		var statusChangeHandle = function(err,status){
			if(err){
				// err.code | err.msg
				return;
			};  
		  
			switch(status){  
				case "not_login" :
					// do something
					break;
				case "logined" :
					// do something
					break;
				case "doing_login" :
					// do something
					break;
				default:
					// do something
			}
		}
		
		
		sumeru.auth.on('statusChange',statusChangeHandle);
		// 完整的调用方式
		sumeru.auth.login('userName','pwd',{a:100,b:200,c:[1,2,3]},'authMehtod_XXX');
		
		// 也可使用以下不完整的参数调用方式。
		// 1.省略authMethod认为 authMethod = 'local'
		// sumeru.auth.login('userName',’pwd‘,{a:100,b:200});
		
		// 2.同时省略args与authMethod时，认为args={},authMethod='local'
		// sumeru.auth.login('userName','pwd');

##sumeru.auth.logout()

登出,无参数.并触发statusChange变化

##sumeru.auth.getStatus()

取得当前的认证状态. 返回值为String，可能的值为：

+ “not_login”	当前未登陆
+ “logined”  已登陆
+ “doing_login” 正在登陆

##sumeru.auth.getLastError()

取得最后一个操作发生的错误信息,每一个新操作产生时,上一次的错误信息将被清空.

> 为了便于识别错误的发生原因及发生位置，认证系统返回的错误信息对像内容，都使用相同的结构和预定义好的错误号码，具体可以参见 “sumeru/src” 目录下的 “COMMON_STATUS.js” 中的具体定义，对于第三方登陆模块产生的错误消息，也要求在这里先进行定义，便于查阅。

EXAMPLE CODE:


		// 在未登陆情况下
		sumeru.auth.getStatus() == 'no_login';	  // true;
		
		//如果登陆错误时，如使用错误的密码等情况，login(user,fakePwd);
		sumeru.auth.getStatus() == 'no_login';	  // true;
		
		// 在未进行其它操作之前，如再次登陆，注册等操作，可以取得上一步登陆错误的信息
		var errObj = sumeru.auth.getLastError();
		
		console.log(err)		// {code:1001,msg:'INVALID USERNAME OR PWD'}


##sumeru.auth.getUserInfo()

取得当前认证用户的信息,如果未登陆则只能拿到null，
	
userInfo对像结构如下

		userInfo = {  
			"token":"token",  
			"info":{"param1":"modify1","param2":"modify2","param3":["1","2"]},  
			"status":"online",  
			"smr_id":"5253a5aa546610001200014a",  
			"__clientId":"7qgj3s1grr",  
			"userId":"5253a5a95466100012000148",  
			"clientId":"183ou3qrg_QZKUYDxapKFO",  
			"authMethod":"local",  
			"expires":1381213910081  
		}

> 其中info字段的内容，来源于注册时提供的userInfo, 参见: [sumeru.auth.register](#sumeruauthregistertokenpwduserinfo-authmethod-callback)

##sumeru.auth.isLogin()  :  *Deprecated*

判断当前的认证状态是否为已登陆，只有在当前的登陆状态为’logined‘的时候返回true，其它时候返回false。

> 这个方法，将被sumeru.auth.getStatus() 进行替代，并将在下一个版本中移除对这个方法的支持。

##sumeru.auth.getToken()  :  *Deprecated*


取得当前的登陆用户的token,未登陆则什么也拿不到.

> 这个方法，将被 sumeru.auth.getUserInfo() 替代，并将在下一个版本中移除对这个方法的支持。

##sumeru.auth.registerValidate(userInfo,authMethod,callback)

测试一个注册信息是否可用. 
 
**userInfo** : object : 待测试的注册信息，  
**authMethod** : string: 用户类型  
**callback** : function : 修改完成后的回调方法,可接收一个err与isUsefull参数,当产生错误是传入对应的错误对像,如果可能使用,则为nul,当isUserfull为true里，由用户信息可用。

> 每个authMethod所要求的userInfo对像内容可能并不相同（取决于对应authMethod的相关实现，参见：*第三方认证扩展* 

----
> 这个方法的需要目标 authMethod 的实现提供支持，一些第三方法登陆过程的实现可能不包含这个方的实现，对于不实现的情况，将返回错误对像 {code:1000,msg:'METHOD UNSUPPORTED'}  

EXAMPLE CODE:

		sumeru.auth.registerValidate({token:'user123',age:100},'local',function(err,isUsefull){
			if(isUserfull){
				// 注册信息验证成功，可以进行注册
				sumeru.auth.register('user123','pwd',{age:100},local,function(err){
					if(err){
						// 注册失败
						return；
						
					}
					// 注册成功
					// do something ..
				});
			}else{
				// 注册信息验证失败
				//err.code || err.msg
			}
		})


##sumeru.auth.register(token,pwd,userInfo, authMethod, callback)


注册一个用户. 


**token** : string : 用户的登陆标识  
**pwd** : string : 登陆密码,  
**userInfo** : object : 新的用户信息对像.  
**authMethod** : string : 目标的用户类型  
**callback** : function : 修改完成后的回调方法,可接收一个err参数,当产生错误是传入对应的错误对像,如果成功,则为null.  

> 在注册前，尽量使用sumeru.auth.registerValidate() 进行验证，可以有效减少错误发生。

EXAMPLE CODE:

参见：[sumeru.auth.registerValidate](#sumeruauthregistervalidateuserinfoauthmethodcallback)

> 这个方法的需要目标authMethod的实现提供支持，一些第三方法登陆过程的实现可能不包含这个方的实现，对于不实现的情况，将返回错误对像 {code:1000,msg:'METHOD UNSUPPORTED'}

##sumeru.auth.modifyUserInfo(token,pwd,userInfo,authMethod,callback)

修改用户信息,
**token** :string : 用户的登陆标识  
**pwd** : string : 登陆密码,  
**userInfo** : object : 新的用户信息对像.  
**authMethod** :string : 目标的用户类型  
**callback** : function : 修改完成后的回调方法,可接收一个err参数,当产生错误是传入对应的错误对像,如果成功,则为null.  

> 这个方法的需要目标authMethod的实现提供支持，一些第三方法登陆过程的实现可能不包含这个方的实现，对于不实现的情况，将返回错误对像 {code:1000,msg:'METHOD UNSUPPORTED'}

##sumeru.auth.modifyPassword(token,oldPwd,newPwd,authMethod,callback)

修改用户登陆密码信息,
**token** : string : 用户的登陆标识  
**oldPwd** : string : 当前使用的登陆密码,  
**newPwd** : string: 新的登陆密码  
**authMethod** : string : 目标的用户类型  
**callback** : function : 修改完成后的回调方法,可接收一个err参数,当产生错误是传入对应的错误对像,如果成功,则为null.  

> 这个方法的需要目标authMethod的实现提供支持，一些第三方法登陆过程的实现可能不包含这个方的实现，对于不实现的情况，将返回错误对像 {code:1000,msg:'METHOD UNSUPPORTED'}


