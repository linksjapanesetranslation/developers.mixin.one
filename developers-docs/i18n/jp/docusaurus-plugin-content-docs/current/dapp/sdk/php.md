---
title: PHP
sidebar_position: 14
---

# PHP SDK

本SDKはComposerをベースにしており、動作にはPHPバージョン7.0以降が必要です。

### インストール

```
$ composer require exinone/mixin-sdk-php -vvv
```

### クイックスタート

```php
<?php

require 'vendor/autoload.php';

$appConfig = [
'mixin_id' => '',
'client_id' => '',
'client_secret' => '',
'pin_token' => '',
'session_id' => '',
'private_key' => <<<EOF
EOF
];

$mixinSdk = new \ExinOne\MixinSDK\MixinSDK();

$mixinSdk->setConfig('appConfig', $appConfig);

// Create a user, and register him on the Mixin Network.
$user = $mixinSdk->use('appConfig')->network()->createUser('fullname');
var_dump($user);

$userConfig = [
'client_id' => $user['user_id'],
'session_id' => $user['session_id'],
'pin_token' => $user['pin_token'],
'private_key' => $user['priKey']
];

$mixinSdk = new \ExinOne\MixinSDK\MixinSDK();
$mixinSdk->setConfig('userConfig', $userConfig);

// Set initial PIN.
$res = $mixinSdk->use('userConfig')->pin()->updatePin('', '123456');
var_dump($res);

// Read user info.
$self = $mixinSdk->use('userConfig')->user()->readProfile();
var_dump($self);

// Get asset list.
$assets = $mixinSdk->use('userConfig')->wallet()->readAssets();
var_dump($assets);

?>
```

その他の例は、[examples](https://github.com/ExinOne/mixin-sdk-php/tree/master/tests/Feature)をご覧ください。

---
本SDKは、Exinチームによって開発されました。テクニカルサポートへのお問い合わせは、Mixin Messengerで26930を検索してください。
