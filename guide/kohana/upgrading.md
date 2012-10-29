<div class="original-doc">
# Migrating from 3.2.x

## HVMC Isolation

HVMC Sub-request isolation has been improved to prevent exceptions leaking from this inner to the outer request. If you were previously catching any exceptions from sub-requests, you should now be checking the [Response] object returned from [Request::execute].
</div>

# 3.2.xからの移行

## HVMC分離
HVMCサブリクエスト分離は内部リクエストから外部リクエストへの例外発生漏れを防ぐように改善されました。

<div class="original-doc">
## HTTP Exceptions

The use of HTTP Exceptions is now encouraged over manually setting the [Response] status to, for example, '404'. This allows for easier custom error pages (detailed below);

The full list of supported codes can be seen in the SYSPATH/classes/HTTP/Exception/ folder.

Syntax:

    throw HTTP_Exception::factory($code, $message, array $variables, Exception $previous);

Examples:

    // Page Not Found
    throw HTTP_Exception::factory(404, 'The requested URL :uri was not found on this server.', array(
            ':uri' => $this->request->uri(),
        ));

    // Unauthorized / Login Requied
    throw HTTP_Exception::factory(401)->authenticate('Basic realm="MySite"');

    // Forbidden / Permission Deined
    throw HTTP_Exception::factory(403);
</div>

## HTTP例外

HTTP例外の使用について現在のところは [Response]ステータス（例えば'404'）を手動で設定することが推奨されます。こうすることによって（以下に記述されるような）簡易なカスタムエラーページを許可することができます。
サポートされているコードの全リストはSYSPATH/classes/HTTP/Exception/フォルダーで確認できます。

Syntax:

    throw HTTP_Exception::factory($code, $message, array $variables, Exception $previous);

Examples:

    // Page Not Found
    throw HTTP_Exception::factory(404, 'The requested URL :uri was not found on this server.', array(
            ':uri' => $this->request->uri(),
        ));

    // 未認証 / ログイン要求
    throw HTTP_Exception::factory(401)->authenticate('Basic realm="MySite"');

    // 参照禁止 / 権限不足
    throw HTTP_Exception::factory(403);



<div class="original-doc">
## Redirects (HTTP 300, 301, 302, 303, 307)

Redirects are no longer issued against the [Request] object. The new syntax from inside a controller is:

    $this->redirect('http://www.google.com', 302);

or from outside a controller:

    HTTP::redirect('http://www.google.com', 302);


Custom error pages are now easier than ever to implement, thanks to some of the changes brought about by the HVMC and Redirect changes above.

See [Custom Error Pages](tutorials/error-pages) for more details.

</div>
## リダイレクト (HTTP 300, 301, 302, 303, 307)

## Custom Error Pages (HTTP 500, 404, 403, 401 etc)
[Request] オブジェクトに対してredirectメソッドを要求することはできなくなりました。
新しい使用方法は以下のようになります。

コントローラーの内部においては:

    $this->redirect('http://www.google.com', 302);

あるいはコントローラーの外部からは:

    HTTP::redirect('http://www.google.com', 302);


HMVCとリダイレクト処理に対する変更のお陰で、カスタムエラーページは以前より簡単に実装できるようになりました。
詳細は[Custom Error Pages](tutorials/error-pages)を参照してください。


<div class="original-doc">
## Browser cache checking (ETags)

The Response::check_cache method has moved to [HTTP::check_cache], with an alias at [Controller::check_cache]. Previously, this method would be used from a controller like this:

    $this->response->check_cache(sha1('my content'), Request $this->request);

Now, there are two options for using the method:

    $this->check_cache(sha1('my content'));

which is an alias for:

    HTTP::check_cache($this->request, $this->response, sha1('my content'));
</div>
## ブラウザキャッシュのチェック (ETag)
Response::check_cache メソッドは[Controller::check_cache]エイリアスとともに[HTTP::check_cache]に移動しています。
以前、このメソッドはコントローラーからこのように使用されていました。

    $this->response->check_cache(sha1('my content'), Request $this->request);

現在このメソッドを使用するには2つの方法があります。

    $this->check_cache(sha1('my content'));

エイリアスを使用する場合は:

    HTTP::check_cache($this->request, $this->response, sha1('my content'));


<div class="original-doc">
## PSR-0 support (file/class naming conventions)

With the introduction of [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) support, the autoloading of classes is case sensitive. Now, the file (and folder) names must match the class name exactly.

Examples:

    Kohana_Core

would be located in

    classes/Kohana/Core.php

and

    Kohana_HTTP_Header

would be located in

    classes/Kohana/HTTP/Header.php

This also affects dynamically named classes such as drivers and ORMs. So for example, in the database config using `'mysql'` as the type instead of `'MySQL'` would throw a class not found error.
</div>
## PSR-0サポート(fileとclassの命名規則)
[PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md)の序章は、クラス名の大文字小文字を区別するオートローディングをサポートします。
現在、ファイル（とフォルダ）名はクラス名と正確に一致する必要があります。

例:

    Kohana_Core

クラスは以下のファイル名であるべきです。

    classes/Kohana/Core.php

そして

    Kohana_HTTP_Header

クラスは以下のファイル名であるべきです。

    classes/Kohana/HTTP/Header.php

<div class="original-doc">
## Query Builder Identifier Escaping

The query builder will no longer detect columns like `COUNT("*")`. Instead, you will need to use `DB::expr()` any time you need an unescaped column. For example:

    DB::select(DB::expr('COUNT(*)'))->from('users')->execute()
</div>
## クエリ・ビルダーのはエスケープ識別

クエリ・ビルダーはもはや`COUNT("*")`のようなカラムを検知しません。
代わりにカラムをエスケープするときにはいつでも`DB::expr()`を使用する必要があります。

<div class="original-doc">
## Route Filters

In `3.3.0`, you can no longer pass a callback to `Route::uri()`. Instead, we've added the ability to define one or more filters which will be able to decide if the route matches and will also allow you to change any of the parameters. These filters will receive the `Route` object being tested, the currently matched `$params` array, and the `Request` object as the three parameters.

    Route::set('route-name', 'some/uri/<id>')
        ->filter(function($route, $params, $request) {
            // Returning FALSE will make this route not match
            // Returning an array will replace the $params sent to the controller
        });

These filters can be used for things like prepending the request method to the action, checking if a resource exists before matching the route, or any other logic that the URI alone cannot provide. You can add as many filters as needed so it's useful to keep filters as small as possible to make them reusable.

See [Routing](routing#route-filters) for more details.
</div>
## ルート・フィルタ
`3.3.0`では`Route::uri()`にコールバックを渡すことはできません。
代わりにルートにマッチするかを判定し且つそのパラメータを変更できる複数のフィルタを定義する機能を追加しました。これらのフィルタはテストされている`Route`オブジェクトが受け取るでしょう。
一致した`$params`の配列と3つのパラメータとして`Request`オブジェクト

    Route::set('route-name', 'some/uri/<id>')
        ->filter(function($route, $params, $request) {
            // このルートにマッチしなければFALSEが返る
            // コントローラーに送られる$paramsを置き換えた配列が返る
        });

これらのフィルタはアクションへのリクエストメソッドを模倣し、ルートにマッチさせる前にリソースが存在するか、あるいはURIが単独で提供できない他のロジックをチェックするようなことに使えます。
フィルタは必要なだけいくらでも追加できます。従ってそれらの再利用を可能にするためフィルタを可能な限り小さく保つことが役に立ちます。
詳細は[Routing](routing#route-filters) を参照してください。
