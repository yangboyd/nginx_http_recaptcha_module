
# nginx_http_recaptcha_module - support google's reCAPTCHA with Nginx

## Description

This module can be deployed in spam or DDOS attack protection for Nginx. It's used the reCAPTCHA to distinguish between human and auto script. The module works with these steps below:

1. The request comes from client. If the request contains the correct secure cookie, it will do the normal action. If not, the request will redirect to the recaptcha page.

2. The client inputs the captcha letters.

3. Nginx sends this input letters to recaptcha server for verification.

4. The correct answer from reccaptcha server with beginning of "true...", else it's beginning with "false...".

5. Add the secure cookie for the correct verified client, redirect the client to the page which he wants to view.

## Getting started

1. Install this module

2. [Get a pair of recaptcha key from google](https://www.google.com/recaptcha/admin/create)

3. Copy the template recaptcha page from captcha_html/captcha.html to your web directory.

4. Replace the public key in the recaptcha page.

5. Replace the private key in the config file.

6. Change the secure_cookie_md5's private key in the config file.

7. Change the domain of yourhost.com to your real domain.

## Examples

```
location / {
    secure_cookie $cookie_CAPTCHA_SESSION,$cookie_CAPTCHA_EXPIRES;
    secure_cookie_md5 private_key$binary_remote_addr$cookie_CAPTCHA_EXPIRES;

    if ($cookie_CAPTCHA_SESSION = "") {
        rewrite ^.*$ /captcha.html redirect;
    }

    if ($cookie_CAPTCHA_EXPIRES = "") {
        rewrite ^.*$ /captcha.html redirect;
    }

    if ($secure_cookie = "0") {
        rewrite ^.*$ /captcha.html redirect;
    }

    if ($secure_cookie = "") {
        return 403;
    }

    proxy_pass http://your_backend;
}

location = /captcha.html {
    root html;
}

location = /verify {
    eval_inherit_body on;
    eval_override_content_type 'text/plain';

    eval $verify_content {
        recaptcha_challenge_name $recaptcha_challenge_field;
        recaptcha_response_name $recaptcha_response_field;

        proxy_method POST;
        proxy_set_header  Accept-Encoding  "";
        proxy_set_body "privatekey=your_privatekey_from_google&remoteip=$remote_addr&challenge=$recaptcha_challenge_field&response=$recaptcha_response_field";

        rewrite .* /recaptcha/api/verify break;
        proxy_pass 'http://www.google.com';
    }

    if ($verify_content ~* ^true[\s\R]*(.*)) {
        set $error_code $1;

        rewrite .* /set_secure_cookie last;
    }

    if ($verify_content ~* ^false[\s\R]*(.*)) {
        set $error_code $1;

        return 403;
    }

    return 404;
}

location = /set_secure_cookie {
    internal;
    secure_cookie_expires 1h;
    secure_cookie_md5 private_key$binary_remote_addr$secure_cookie_set_expires_base64;

    add_header Set-Cookie "CAPTCHA_SESSION=$secure_cookie_set_md5; expires=$secure_cookie_set_expires; path=/; domain=.yourhost.com";
    add_header Set-Cookie "CAPTCHA_EXPIRES=$secure_cookie_set_expires_base64; expires=$secure_cookie_set_expires; path=/; domain=.yourhost.com";

    rewrite ^.*$ http://www.yourhost.com redirect;

    return 302;
}
```

## Directives

### recaptcha_challenge_name

    syntax: *recaptcha_challenge_name
    $variable_stored_content_of_recaptcha_challenge_field;*

    default: *none*

    context: *http, server, location*

    description: The name should equal to the name of challenge input form.
    This directive will add the specific variable. This variable is used
    only in the directive of proxy_set_body. It will get the value of the
    challenge input form. It's equal to "$recaptcha_challenge_field"
    generally.

### recaptcha_response_name

    syntax: *recaptcha_response_name
    $variable_stored_content_of_recaptcha_response_field;*

    default: *none*

    context: *http, server, location*

    description: The name should equal to the name of response input form.
    This directive will add the specific variable. This variable is used
    only in the directive of proxy_set_body. It will get the value of the
    response input form. It's equal to "$recaptcha_response_field"
    generally.

## Installation

Download the [latest version of the release tarball of nginx_eval_module](https://github.com/yaoweibin/nginx-eval-module).

Download the [latest version of the release tarball of nginx_secure_cookie_module](https://github.com/yaoweibin/nginx_secure_cookie_module).

Download the [latest version of the release tarball of this module](https://github.com/yaoweibin/nginx_http_recaptcha_module).

Grab the [nginx source code](http://nginx.org/), for example, the version 0.8.54 (see nginx compatibility), and then build the source with this module:

    $ wget 'http://nginx.org/download/nginx-0.8.54.tar.gz'
    $ tar -xzvf nginx-0.8.54.tar.gz
    $ cd nginx-0.8.54/

    $ ./configure --add-module=/path/to/nginx_http_recaptcha_module \
        --add-module=/path/to/nginx_secure_cookie_module \
        --add-module=/path/to/nginx_eval_module

    $ make
    $ make install

## Compatibility

My test bed is 0.8.54.

## TODO

### Known Issues

*   Developing

*   Google limits 1 million reCAPTCHA requests per day for each key. [See faq](http://www.google.com/recaptcha/faq).

*   If you use the global key for many sites, you should not add the domain field in the Set-Cookie header.

## Changelogs

### v0.1

first release

## Authors

Weibin Yao(姚伟斌) *yaoweibin at gmail dot com*

## Copyright & License

This README template copy from [agentzh](http://github.com/agentzh).

This module is licensed under the BSD license.

Copyright (C) 2010 by Weibin Yao <yaoweibin@gmail.com>.

All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are
met:

*   Redistributions of source code must retain the above copyright
    notice, this list of conditions and the following disclaimer.

*   Redistributions in binary form must reproduce the above copyright
    notice, this list of conditions and the following disclaimer in the
    documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED
TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A
PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED
TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

