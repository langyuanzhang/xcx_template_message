# xcx_template_message
微信小程序模板消息




```
//测试模板消息
public function commithf($pid,$nickname,$content){
    //任务完成通知 start

    $commit = Db::table('commit')->where('id','eq',$pid)->find();
    $formid = $commit['formid'];
    $user_id = $commit['user_id'];

    $recruitmeid = $commit['recruitmeid'];
    $merchantsid = $commit['merchantsid'];

    $user = User::where('id','eq',$user_id)->find();
    $openid = $user['openid'];

    $addtime = date('Y-m-d H:i:s',time()); //当前时间

    $template_id = 'OjhExCpuLsOFdVDXHtiSfRlR4vryMn5bx5hJs9kx5vY'; //模板id

    $time=date('Y-m-d H:i:s',time()); 

    $datainfo  = [
    'keyword1'=>[
        'value'=>"xxx提示",
        'color'=>'#000000'
    ],
    'keyword2'=>[
        'value'=>"'".$nickname."'",
        'color'=>'#173177'
    ],
    'keyword3'=>[
        'value'=>"“".$content."”",
        'color'=>'#173177'
    ]
    ,
    'keyword4'=>[
        'value'=>$time,
        'color'=>'#173177'
    ],
    ];

    $page = 'pages/recruitment/seemore/seemore?recruitmeid=1';

    $res = $this->send_template($openid,$template_id,$formid,$datainfo,$user_id,$page);

    //任务完成通知end
}
```

```
//发送模板消息
/**
*@param $applet_id 小程序ID
*@param $user_id  用户ID
*@param $form_id  表单ID
*@param  $data     消息模板
*/
public function send_template($openid,$template_id,$formid,$datainfo,$user_id,$page){
    //form_id 表单提交场景下，为submit事件带上的formId；支付场景下，为本次支付的prepay_id
    //获取token值

    $config = Db::table('wx_config')->where('id','eq','1')->find();

    $url="https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=".$config['x_appid']."&secret=".$config['x_appsecret'];

    $result=$this->curl_get($url);

    $access_token=$result['access_token'];

        //拼装URL
    $url="https://api.weixin.qq.com/cgi-bin/message/wxopen/template/send?access_token=".$access_token;

        //拼装模板消息
    $template_data=[
        'touser'=>$openid,
        'template_id'=>$template_id,
        'page'=>$page,
        'form_id'=>$formid,
        'data'=>$datainfo,
        'emphasis_keyword'=>'',
    ];
    $template_data=json_encode($template_data);

    $result = $this->curl_post_https($url,$template_data);

    return $result;
}
```

```
//get请求
public function curl_get($url){
    $curl = curl_init();
    curl_setopt($curl, CURLOPT_URL, $url);
    curl_setopt($curl, CURLOPT_HEADER, 0);
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
    curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);//这个是重点。
    curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, FALSE);
    $data = curl_exec($curl);
    curl_close($curl);
    $data=json_decode($data,true);
    return $data;
}
/* PHP CURL HTTPS POST */
public function curl_post_https($url,$data){ // 模拟提交数据函数
    $curl = curl_init(); // 启动一个CURL会话
    curl_setopt($curl, CURLOPT_URL, $url); // 要访问的地址
    curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, 0); // 对认证证书来源的检查
    curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, 2); // 从证书中检查SSL加密算法是否存在
    curl_setopt($curl, CURLOPT_USERAGENT, $_SERVER['HTTP_USER_AGENT']); // 模拟用户使用的浏览器
    curl_setopt($curl, CURLOPT_FOLLOWLOCATION, 1); // 使用自动跳转
    curl_setopt($curl, CURLOPT_AUTOREFERER, 1); // 自动设置Referer
    curl_setopt($curl, CURLOPT_POST, 1); // 发送一个常规的Post请求
    curl_setopt($curl, CURLOPT_POSTFIELDS, $data); // Post提交的数据包
    curl_setopt($curl, CURLOPT_TIMEOUT, 30); // 设置超时限制防止死循环
    curl_setopt($curl, CURLOPT_HEADER, 0); // 显示返回的Header区域内容
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1); // 获取的信息以文件流的形式返回
    $tmpInfo = curl_exec($curl); // 执行操作
    if (curl_errno($curl)) {
        echo 'Errno'.curl_error($curl);//捕抓异常
    }
    curl_close($curl); // 关闭CURL会话
    return $tmpInfo; // 返回数据，json格式
}
```