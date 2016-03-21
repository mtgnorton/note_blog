<h1 id="正方系统模拟登陆">正方系统模拟登陆</h1>
<p>在爬取的时候，首先使用浏览器登陆进行抓包，<br>
<img src="https://i.imgur.com/wTfFiMW.png" alt="enter image description here"></p>
<p>第一次抓包主要提交了两次，第一次是向default2.aspx进行post提交<br>
<img src="https://i.imgur.com/6d3i0l3.png" alt="enter image description here"></p>
<p>，虽然被重定向，但是这一次提交，完成了数据的验证，验证的字段如图所示，第一个字段是登陆界面的一个隐藏字段，可以通过正则获取到，txtSecretCode是验证码，这是正方系统的一个小bug，在我们模拟登陆的时候可以忽略，RadioButtonList1代表的是学生。<br>
<img src="https://i.imgur.com/VUfXeXz.png" alt="enter image description here"><br>
上图中我们可以看到Request URL中附加了一个字段，这个字段是随即的，经过测试，得出正方系统有两种登陆方式，一种是通过cookie来确认用户，另一种是通过随即字符串的形式，现在我们校网所使用的是随机字符串的形式。在第一次向default2.aspx 进行post提交时，可能是通过js产生了随即字符串并被附着在url中，验证通过之后会被重定向到xs_main.aspx页面,访问此页面通过的是get方式。<br>
<img src="https://i.imgur.com/cQl9Buk.png" alt="enter image description here"><br>
此时的字符串和上次post提交时的是相同的。<br>
分析完成之后，代码如下</p>
<pre><code>    function curl_request($url,$post='',$referer=''){
        $curl = curl_init();
        curl_setopt($curl, CURLOPT_URL, $url);
        curl_setopt($curl, CURLOPT_USERAGENT, 'Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)');
        curl_setopt($curl, CURLOPT_FOLLOWLOCATION, 1);
        curl_setopt($curl, CURLOPT_AUTOREFERER, 1);
        curl_setopt($curl, CURLOPT_HEADER, 0);
        curl_setopt($curl, CURLOPT_REFERER, "http://211.87.155.19/default2.aspx");
        if($post) {
            curl_setopt($curl, CURLOPT_POST, 1);
            curl_setopt($curl, CURLOPT_POSTFIELDS, http_build_query($post));
        }
    
        if ($referer) {
             curl_setopt($curl, CURLOPT_REFERER, $referer);
                      }
      else{
                curl_setopt($curl, CURLOPT_HEADER, 1);
          }

        
        curl_setopt($curl, CURLOPT_TIMEOUT, 10);
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
        $data = curl_exec($curl);
        curl_close($curl);
     
}
</code></pre>
<p>这是自己写的一个curl函数，用来完成基本的操作。</p>
<pre><code>    function getView(){
     $res;
     $url = 'http://211.87.155.19/default2.aspx';
     $result = $this-&gt;curl_request($url);
     preg_match('/Location: \/\((.*)\)/', $result,$aa);
     $pattern = '/&lt;input type="hidden" name="__VIEWSTATE" value="(.*?)" \/&gt;/is';
     preg_match_all($pattern, $result, $matches);
     $res[0] = $matches[1][0];
     $res[1] = $aa[1];
     return $res;
}
</code></pre>
<p>这段代码的作用是获得__VIEWSTATE和url中的随机字符串。</p>
<pre><code> function login($temp){ 
     

     $url = 'http://211.87.155.19/('.$temp[1].')/default2.aspx';

     $post['__VIEWSTATE'] = $temp[0];
     $post['txtUserName'] = '';//你的账号
     $post['TextBox2'] = '';//你的密码
     $post['txtSecretCode'] = '';
     $post['lbLanguage'] = '';
     $post['RadioButtonList1'] =iconv('utf-8', 'gb2312', '学生');
     $post['Button1'] = '';
     $result = $this-&gt;curl_request($url,$post);//模拟第一次post提交
     $referer='http://211.87.155.19/('.$temp[1].')/xs_main.aspx?xh='.$post['txtUserName'];
     $result =$this-&gt;curl_request($referer,'',$url);//模拟第二次get提交
    
     }
</code></pre>
<pre><code>		$temp = getView();
	    login($temp);

</code></pre>
<p>此时即完成模拟登陆</p>
