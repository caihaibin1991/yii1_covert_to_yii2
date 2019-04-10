# yii1_covert_to_yii2
yii1代码简单转化成yii2, 根据自己框架的特点,可实现约9成的代码自动替换实现

模块自动转化工具

/yii2sys/_cover_tools/replacefile_to_yii2.php
通过配置指定目录,脚本可实现yii1绝大部分自动转换到yii2形式
主要区别
yii2补充命名空间
工具已实现页面和控制器,自动引入models相关类,model类和控制器自动追加命名空间
全局类引入,非常用类型,直接以\yii\xxx 或\ext\xxx形式带入,减少不必要引入代码
rbac
基本和yii1相同,但是一些数据字段名做了调整(如:adminid调整为admin_id,很多类似字段都补充了下划线)
ar的查询方式
通过find的形式返回ActiveQuery对象,通过对该对象进行操作实现查询
原 CDbCriteria 对象被取消,工具已实现常用查询类的自动转化,关联查询仍然需要进行手动补充调整.
关联查询,通过配置查询对象,实现分页,主要特殊在主表和子表重命名,具体看项目案例.
分页对象做了调整,默认$_GET[‘page’]不做为默认分页值,需要手动设置
系统结构调整
/system 原模块被整合到/app/modules/里
框架结构
/yii2framework
使用yii2高级模板,已做结构的调整同时项目yii2sys也做了调整,使现有结构形式按原来的部署方式
(注:有需要可自行下载原高级模板进行了解)
编辑器使用
为实现快速开发,所有有ar返回的函数,函数注释可以通过补充@return ActiveRecord 实现代码提示(需要在inlucde_path中引入框架做为附属项目)
主要修改点整理:
url() yii2fix module 默认会到根当成模块
导致顶根判断异常,所以需要跳过
自定义state
yii2已移除state,官方说法是避免混乱.导致无法同域名多应用session共享冲突
md5版本
可以实现端口共享session
域名+目录+程序id版本
实现多项目时,自动区分session数据
\Wechat 类第三方重写,可能类很多,编写时,统一全部认根域名.实现快速重写
HeadersAlreadySentException 异常
// ini_set ( ‘output_buffering’, 0 );设置无效,还是需要通过修改配置.不然超出会抛
__CLASS__ 会带入命名空间路径
php 7.2 下 upload 上传兼容 media data missing hint 异常
兼容方式为 curl_file_create 形式
Yii2 已 去除任意参数 路径化处理.
需要路径化的参数,只能通过配置实现. 前台除了控制器经过重写外,其它参数全部都以 ? 形式拼接
\yii\helpers\Html::DropDownList 不再自动代入id,需要通过htmloptions配置进去
CHtml 调整成 Html
CWenhaoPager.php 重写成 LinkPager 形式
反射类调用,需要写全命名空间
计划任务
表前缀 {{%table_name}} 多了个 “%”
入口环境 测试环境(保留) 开发环境 ,生产环境 切换
将/system 移动到 app/modules/system 做为 app 下的模块
Yii::$app->createUrl 换成 Yii::$app->urlManager->createUrl
CDbCriteria
$models = (new \app\modules\system\models\Menu ())->find ()->where ( “parentId=0 and state=1” )->orderBy ( ‘ordernum asc,id asc’ )->all ();
ar 修改成yii2形式,需要实现接口继承+覆盖几个内置函数
$model->getAttributes ( false ); false 去掉 ,默认动作只识别null
Yii::$app->request->isPostRequest 替换成 Yii::$app->request->post ()
Yii::$app->controller->getAction ()->getId () Yii::$app->controller->action->id
Yii::$app->controller->getId () Yii::$app->controller->id
Yii::$app->controller->getModule () Yii::$app->controller->module->id
controllers 控制器 render 类动作,需要返回. 变成 return $this->render…
enableCsrfValidation 全局.暂时csrf校验关闭,默认开启.动作在action之前
后台基本版 igd_system_adminauthassignment 表 userid 调整为 user_id (框架写死字段)
beforeAction 不能直接返回true,需要调用父函数,否则会导致过滤器无法触发
数据库权限判断 app ()->authManager->checkAccess ( $user_id, $permission ) 参数顺序变化
表重命名
AdminLog::model ()->find ()->alias ( ‘t’ );
表链接
AdminLog::model ()->find ()->innerJoinWith ( [
// 关联表,重命名
‘admin as a’
] )
http 异常抛出调整
throw new CHttpException ( 404, ‘The ‘ . __CLASS__ . ‘ does not exist.’ );
throw new \yii\web\HttpException ( 404, ‘The ‘ . __CLASS__ . ‘ does not exist.’ );
动态表限制实在太多.后续有需要,需要通过ArticleX->fixTableName进行修正
原find(where)调整成find()->where(where)->one()
原findAll(where)调整成find()->where(where)->all()
