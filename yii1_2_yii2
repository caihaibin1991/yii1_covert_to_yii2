只针对模块形式代码进行yii1到yii2的转化,config中需要手动进行补充配置.
<?PHP
header ( 'Content-Type: text/html; charset=utf-8' );
include 'FileTools.php';
include 'lib.php';
// 需要转换的模块
$path = 'D:/PHPnow-1.5.6/htdocs/igddata/yii2sys/app/modules/member/';
// $path = 'D:/PHPnow-1.5.6/htdocs/igddata/yii2sys/app/';
// 根路径,用于识别命名空间
$root_path = 'D:/PHPnow-1.5.6/htdocs/igddata/yii2sys/';
$root_path = preg_replace ( '/\//', '\\', $root_path );
$preg_file = array ();
// Yii::$app->user->setState('_id',$identity->_id); 修改 成 Yii::$app->session->set ( '_id', $identity->getId () );
// ar getList findAll 需要针对性处理.
// CHTML
// <?
$preg_contents = array (
		'/Yii\:\:app\s*?\(\)/' => 'Yii::$app',
		// $app->params->membermodules => 变成数组 ['membermodules']
		'/Yii\:\:\$app\-\>params\-\>(\w+)/' => 'Yii::$app->params[\'\\1\']',
		// 直接调用\Yii跳过use
		'/(?<!\\\\)Yii/' => '\Yii',
		// CHTML 全部调整成 \yii\helpers\Html
		'/(?<!\\\\)CHTML/ims' => '\yii\helpers\Html',
		// 所有短标签自动补充成<?php
		'/\<\?(\s)/' => '<?php\\1',
		// post请求判断切换成yii2形式判断
		'/isPostRequest/' => 'isPost',
		'/getIsPostRequest\s*\(\s*\)/' => 'isPost',
		// user->setState 在新版本中已经被取消
		'/user\-\>setState/' => 'session\-\>set',
		// 统一当前登录用户获取用户id
		'/\>\_id/' => '>getId ()',
		// 所有原参数获取,调整为默认获取
		'/getParam\s*\(\s*?([\'\"\w]+)[^\)]*?\)/ims' => '$_REQUEST[\\1]',
		// '/render\s*\(/' => 'renderPartial(',
		// 渲染页面动作需要进行返回
		'/\n(\s*)\$this\-\>render/' => '
\\1return $this->render',
		// 统一当前登录用户获取用户id
		'/user\-\>\_id/' => 'user->getId()',
		// getAttributes()不能带false参数, 不然会导致出错
		'/getAttributes\s*\(\s*false\s*\)/' => 'getAttributes()',
		// ;;去除无效代码,但是for(;;)需要绕过
		'/(?<!\()\;\s*\;/' => ';',
		// 补充表前缀
		'/\{\{(?=\w)/' => '{{%',
		'/findByPk/i' => 'findOne',
		// 所有数组修改成新形式
		'/(?<!\w)array\s*\(([^\(\)]*?)\)/ms' => '[\\1]',
		'/jquery\-easyui\-1\.2\.5/ims' => 'jquery-easyui-1.3.1',
		'/public\s*function\s*tableName/ims' => 'public static function tableName',
		'/getAction\s*\(\)/' => 'action',
		// load为默认函数,不可用该函数名,全部调整为loadModel
		'/([\s\>])load\s*\(/' => '\\1loadModel(',
		// 修改http异常,并直接引用
		'/CHttpException/' => '\yii\web\HttpException',
		// 修改ar继承类
		'/extends CActiveRecord/' => 'extends \yii\db\ActiveRecord',
		// 修改注释,用于引入框架时,实现代码提示
		'/CActiveRecord/' => 'ActiveRecord',
		'/action\-\>getId\s*\(\)/' => 'action->id',
		'/getController\s*\(\)/' => 'controller',
		'/controller\-\>getId\s*\(\)/' => 'controller->id',
		'/getModule\s*\(\)/' => 'module',
		'/module\-\>getId\s*\(\)/' => 'module->id',
		// array 引用
		'/require\_once\s*\(\s*[\'\"]array\.php[\'\"]\s*\)\;/ims' => 'lib(\'array.php\');',
		'/require\_once\s*\(\s*[\'\"]array2\.php[\'\"]\s*\)\;/ims' => 'lib(\'array2.php\');',
		// 去除WebControls引用
		'/require\_once\s*\(\s*[\'\"]WebControls\.php[\'\"]\s*\)\;/' => '',
		// 直接调用WebControls
		'/new WebControls/' => 'new \ext\WebControls',
		// 去除MysqlTools引用
		'/require\_once\s*\([\'\"]MysqlTools\.php[\'\"]\)\;/' => '',
		'/lib\s*\(\s*[\'\"]MysqlTools\.php[\'\"]\s*\)\;/' => '',
		// 直接调用MysqlTools
		'/(?<!\\\\)MysqlTools/' => '\ext\MysqlTools',
		// GCheckData直接转化
		'/(?<!\\\\)GCheckData/' => '\ext\GCheckData',
		// 替换控制器集成类
		'/extends\s*CController/' => 'extends \yii\web\Controller',
		// beforeAction只能为公开函数
		'/protected\s*function\s*beforeAction/ims' => 'public function beforeAction',
		// 判断到有beforeAction的,需要进行父函数调用,否则会导致过滤器失效
		'/beforeAction/' => function ($path, $content) {
			// 有使用before,但是没有进行父函数调用,需要补充父类beforeAction调用
			if (! preg_match ( '/parent\:\:beforeAction/', $content )) {
				$content = preg_replace ( '/beforeAction[^\{]+?\{/ims', 'beforeAction($action) {
		// 需要调用父函数,否则不会触发过滤器
		if (!parent::beforeAction ( $action )) {
			return false;
		}', $content );
			}
			return $content;
		},
		// 整个替换module.php内容,不然配置文件需要手动修改.
		'/CWebModule/' => function ($path, $content) {
			$namespace = getNamespaceByPath ( $path );
			$content = <<<EOF
<?php
namespace {$namespace};
class Module extends \yii\base\Module {
	public function init() {
		parent::init ();
	}
}
EOF;
			return $content;
		},
		// ar特殊处理,补充命名空间
		// 需要先进行命名空间的添加,用于控制器,页面添加对象时,直接识别命名空间,并添加
		'/\\\\yii\\\\db\\\\ActiveRecord/' => function ($path, $content) {
			$namespace = getNamespaceByPath ( $path );
			// \s和(?!)正向零宽断言会导致误判,需要后面补充一个标志
			if (! preg_match ( '/^\<\?php\s*namespace/is', $content )) {
				$content = preg_replace ( '/^\<\?php\s*/ims', "<?php\nnamespace {$namespace};\n", $content );
			}
			// ArticleX 的model有特殊实现,不进行替换.
			if (! preg_match ( '/ArticleX/', $path ) && preg_match ( '/function\s*model\s*\(/ims', $content )) {
				$content = preg_replace ( '/function\s*model\s*\([^\}]+?\}/ims', 'function model($className = __CLASS__) {
		return new self ();
	}', $content );
			}
			if (preg_match ( '/function\s*getList\s*\(\s*\$criteria\s*\=\s*null/ims', $content )) {
				$content = preg_replace ( '/function\s*getList\s*\(\s*\$criteria\s*\=\s*null\s*/ims', 'function getList($afmodel = null', $content );
				$content = preg_replace ( '/(function\s*getList\s*\(\s*\$afmodel\s*\=\s*null\s*[^\;]+?)\$this\-\>findAll\s*\(\s*\$criteria\s*\)\s*\;/ims', '\\1$afmodel->all();', $content );
			}
			if (preg_match ( '/function\s*getCount\s*\(\s*\$criteria\s*\=\s*null/ims', $content )) {
				$content = preg_replace ( '/function\s*getCount\s*\(\s*\$criteria\s*\=\s*null\s*/ims', 'function getCount($afmodel = null', $content );
				$content = preg_replace ( '/(function\s*getCount\s*\(\s*\$afmodel\s*\=\s*null\s*[^\;]+?)\$this\-\>count\s*\(\s*\$criteria\s*\)\s*\;/ims', '\\1$afmodel->count();', $content );
			}
			return $content;
		},
		// 控制器特殊处理,补充命名空间,修改过滤器
		'/\\\\yii\\\\web\\\\Controller/' => function ($path, $content) {
			$namespace = getNamespaceByPath ( $path );
			$models_namespace = getNamespaceByPath ( dirname ( $path ) ) . '\\models';
			
			// 修改过滤器
			// 其它特殊模块,照样修改
			if (preg_match ( '/public\s*function\s*filters\(\s*\)\s*\{[^}]+?\}/ims', $content )) {
				$string = <<<EOF
public function behaviors() {
		return [ 
				'access' => [ 
						'class' => \\ext\\RbacFilter::className () 
				] 
		];
	}				
EOF;
				$content = preg_replace ( '/public\s*function\s*filters\(\s*\)\s*\{[^}]+?\}/ims', $string, $content );
			}
			// 先获取ar列表,并添加到use中,再添加命名空间
			// 获取所有ar,并标记到use中和页面中
			$models = getModelsByControllerPath ( $path );
			$use_models = array ();
			foreach ( $models as $model ) {
				$model_ns = $models_namespace . '\\' . $model;
				// 判断是否有该类同时判断是否已添加命名空间
				if (preg_match ( "/\W{$model}\W/ms", $content ) && ! stristr ( $content, $model_ns )) {
					$content = preg_replace ( '/^\<\?php\s*/ims', "<?php\nuse \\{$model_ns};\n", $content );
					$use_models [] = $model;
				}
			}
			// 控制器补充命名空间使用
			// \s和(?!)正向零宽断言会导致误判,需要后面补充一个标志
			// 命名空间前面不能直接加反斜杠,use 前可以
			if (! preg_match ( '/^\<\?php\s*namespace/ims', $content )) {
				$content = preg_replace ( '/^\<\?php\s*/ims', "<?php\nnamespace {$namespace};\n", $content );
			}
			// 页面补充命名空间使用
			$views = getViewsByControllerPath ( $path );
			foreach ( $views as $view ) {
				$page_content = file_get_contents ( $view );
				// 去除头部和尾部空白部分
				$page_content = trim ( $page_content );
				// 如果没有php头,就补充php头
				if (preg_match ( '/^(?!\<\?php\s*)/i', $page_content )) {
					$page_content = preg_replace ( '/^(?!\<\?php\s*)/i', '<?php

?>', $page_content );
				}
				$use_models = array ();
				foreach ( $models as $model ) {
					$model_ns = $models_namespace . '\\' . $model;
					// 判断是否有该类同时判断是否已添加命名空间
					if (preg_match ( "/\W{$model}\W/ms", $page_content ) && ! stristr ( $page_content, $model_ns )) {
						$page_content = preg_replace ( '/^\<\?php\s*/is', "<?php\nuse \\{$model_ns};\n", $page_content );
					}
				}
				file_put_contents ( $view, $page_content );
			}
			return $content;
		},
		// $criteria =new CDbCriteria 及分页逻辑实现自动转化
		'/\$criteria\s*\=\s*new\s*CDbCriteria.+?(\$this|\$model|\w+|\w+\:\:model\s*\(\s*\))\-\>(findAll|find|getList)\s*\([^\;]*?\)\;/ims' => function ($path, $content, $preg) {
			while ( preg_match ( $preg, $content, $matchs ) ) {
				$subcontent = $matchs [0];
				$object = $matchs [1];
				// 如果是xxx::model()和$this ,需要对(:,(,),$)补充斜杠,进行转义处理,用于正则表达式
				if (preg_match ( '/([\:\(\)\$])/', $object )) {
					$preg_object = preg_replace ( '/([\:\(\)\$])/', '\\\\\\1', $object );
				}
				// 调试识别结果
				// gddebug ( $preg_object, $object, $subcontent, $matchs );
				// exit ();
				// 原CDbCriteria编程ActiveQuery形式,直接通过find返回该对象,取代.
				$subcontent = preg_replace ( '/\$criteria\s*\=\s*new\s*CDbCriteria\s*(\(\s*\))?\;/ims', "\$afmodel=(\$afmodel?\$afmodel:{$object}->find());", $subcontent );
				// CDbCriteria condition 转化,全部通过andWhere代替,重叠部分,会自动补充and处理
				$subcontent = preg_replace ( '/\$criteria\s*\-\>condition\s*\.?\s*\=\s*([^\;]+?)\;/ims', "\$afmodel->andWhere(\\1);", $subcontent );
				// CDbCriteria addCondition 转化,全部通过andWhere代替,重叠部分,会自动补充and处理
				$subcontent = preg_replace ( '/\$criteria\s*\-\>addCondition\s*\(([^\;]+?)\)\;/ims', "\$afmodel->andWhere(\\1);", $subcontent );
				// CDbCriteria order 转化
				$subcontent = preg_replace ( '/\$criteria\s*\-\>order\s*\.?\s*\=\s*([^\;]+?)\;/ims', "\$afmodel->orderBy(\\1);", $subcontent );
				// CDbCriteria select 转化
				$subcontent = preg_replace ( '/\$criteria\s*\-\>select\s*\.?\s*\=\s*([^\;]+?)\;/ims', "\$afmodel->select(\\1);", $subcontent );
				// ActiveRecord count 转化
				$subcontent = preg_replace ( '/' . $preg_object . '?\s*\-\>count\s*\([^\;]*?\)\;/ims', "\$afmodel->count();", $subcontent );
				
				// 如果出现特殊自定函数,修改传入参数
				$subcontent = preg_replace ( '/\-\>getCount\s*\(\s*\$criteria\s*\)/', '->getCount($afmodel)', $subcontent );
				
				// CPagination分页对象转化
				$subcontent = preg_replace ( '/new\s*CPagination\s*\(([^\)]+?)\)/ims', 'new \yii\data\Pagination ( [ 
				"totalCount" => \\1
		] )', $subcontent );
				// 分页大小转化
				$subcontent = preg_replace ( '/\$pages\s*\-\>\s*pageSize\s*\=([^\;]+)\;/ims', "\$pages->setPageSize ( \\1 );", $subcontent );
				// $pages->applyLimit($criteria) 没用,替换成page变量设置及分页变量带入ActiveQuery处理
				$subcontent = preg_replace ( '/\$pages\s*\-\>\s*applyLimit([^\;]+)\;/ims', "\$pages->setPage ( \$page-1 );
			\$afmodel->offset ( \$pages->offset )->limit ( \$pages->limit );", $subcontent );
				// ActiveRecord::findAll转化成 ActiveQuery::all()
				$subcontent = preg_replace ( '/' . $preg_object . '\s*\-\>(findAll|find\s*\(\s*\)\-\>)\s*\([^\;]*?\)\;/ims', "\$afmodel->all();", $subcontent );
				// $model->getList进行特殊处理
				$subcontent = preg_replace ( '/' . $preg_object . '\s*\-\>getList\s*\([^\;]*?\)\;/ims', "{$object}->getList(\$afmodel);", $subcontent );
				// 所有andWhere 内多余的and进行剔除
				$subcontent = preg_replace ( '/andWhere\s*\(\s*(\'|\")\s*and/ims', 'andWhere(\\1', $subcontent );
				// 调试转化结果
				// gddebug ( $subcontent );
				// exit ();
				$content = str_replace ( $matchs [0], $subcontent, $content );
			}
			return $content;
		},
		'/(url\(\'\w+\'\)\?\>)\/(\w+)\//ims' => '\\1&\\2=',
		// 原find(where)调整成find()->where(where)->one()
		'/\-\>find\s*\(([^\)]+)\)/ims' => '->find ()->where (\\1)->one ()',
		// 原findAll(where)调整成find()->where(where)->all()
		'/\-\>findAll\s*\(([^\)]+)\)/ims' => '->find ()->where (\\1)->all ()',
		// count 替换成 find()->where()->count();
		'/\-\>count\s*\(([^\)]+)\)/ims' => '->find ()->where (\\1)->count ()',
		// deleteByPk转化成findOne()->delete();
		'/\-\>deleteByPk\s*\(([^\)]+)\)/ims' => '->findOne(\\1)->delete ()',
		// 页面url自动转化成全小写
		'/(?:\W)url\s*\(\s*\'[^\']+\'/ims' => function ($path, $content) {
			$content = preg_replace_callback ( '/url\s*\(\s*\'[^\']+\'/ims', '_strtolower', $content );
			return $content;
		},
		// 控制器,动作,只首字母大写,后半部分全部小写
		'/function\s*action(\w)\s*\(\s*\)/ims' => function ($path, $content) {
			$content = preg_replace_callback ( '/function\s*action(\w+)\s*\(\s*\)/ims', '_ucfirst', $content );
			return $content;
		} 
);
function _strtolower($matches) {
	return strtolower ( $matches [0] );
}
function _ucfirst($matches) {
	$action = strtolower ( $matches [1] );
	// actions 为过滤器配置,不进行过滤处理
	if ($action === 's') {
		return 'function actions()';
	}
	return 'function action' . ucfirst ( $action ) . '()';
}
function getModelsByControllerPath($path) {
	$models_dir = dirname ( dirname ( $path ) ) . '/models';
	$list = FileTools::getCurList ( $models_dir );
	if (! $list)
		return array ();
	$items = array ();
	foreach ( $list as $item ) {
		$filename = basename ( $item );
		$classname = str_replace ( '.php', '', $filename );
		$items [$classname] = $classname;
	}
	return $items;
}
function getViewsByControllerPath($path) {
	$ctlName = basename ( str_replace ( 'Controller.php', '', $path ) );
	$ctlName = strtolower ( $ctlName );
	$models_dir = dirname ( dirname ( $path ) ) . '/views/' . $ctlName;
	$list = FileTools::getCurList ( $models_dir );
	if (! $list)
		return array ();
	$items = array ();
	foreach ( $list as $item ) {
		$filename = basename ( $item );
		$classname = str_replace ( '.php', '', $filename );
		$items [$classname] = $item;
	}
	return $items;
}
function getNamespaceByPath($path) {
	global $root_path;
	$path = dirname ( $path );
	$path = preg_replace ( '/\//', '\\', $path );
	return str_replace ( $root_path, '', $path );
}
// sql 中，pow需要手动检查，替换成 power ,pi 需要手动检查，替换成 acos(-1)

$preg_file = array (
		'/^\w+Module\.php$/' => 'Module.php' 
);
FileTools::replacenames ( $path, $preg_file );
FileTools::replaceContents ( $path, $preg_contents, array (
		'/\.php$/i' => false,
		'/yiic/i' => true 
) );

//直接根据列表文件进行替换规则处理

