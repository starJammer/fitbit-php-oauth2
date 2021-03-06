# Unofficial Fitbit Client Library for PHP using OAuth2

Wholesale borrows large portions of [djchen/OAuth2-Fitbit](https://github.com/djchen/oauth2-fitbit) (minor change to
 error checking and scope handling) and [pavelrisenberg/fitbitphp](https://github.com/pavelrisenberg/fitbitphp).

Sets a fitbit-php-oauth2-state cookie during auth flow to prevent CSRF attacks. A session must be started beforehand.

Not guaranteed to work under any circumstances, but it's nice when it does.

## Installation

To install, use composer:

```composer require brulath/fitbit-php-oauth2```

## Usage

### Warning

I'm lazy, so I've made this library automatically refresh oauth details whenever they've expired mid-call. That means
 after any call the oauth token may have changed, which you will need to check for (and save the new token). I figure
 it's probably easier to check for changed tokens than catching token expiration exceptions and handling those.
 Soz brah.

### Magic token acquisition

```php
try {
    $fitbit = new brulath\fitbit\FitbitPHPOAuth2(
        'your_client_id',
        'your_client_secret',
        'your_post_authorization_redirect_url',
        ['activity', 'heartrate', 'location', 'nutrition', 'profile', 'settings', 'sleep', 'social', 'weight'], // desired scopes
        true  // produce some debugging output in error_log
    );
    
    // A session is required to prevent CSRF
    session_start();
    
    $access_token = $fitbit->getToken();  // will redirect user to fitbit. the cookie it sets must survive.
    
    storeAccessTokenAsJsonInMyDatabase($access_token);
} catch (\Exception $e) {
    print($e);
}
```

### Restoring access
```php

try {
    $fitbit = new brulath\fitbit\FitbitPHPOAuth2(
        'your_client_id',
        'your_client_secret',
        'your_post_authorization_redirect_url',
        ['activity', 'heartrate', 'location', 'nutrition', 'profile', 'settings', 'sleep', 'social', 'weight'], // desired scopes
        true  // produce some debugging output in error_log
    );
    
    // If token has expired, the first request you make will additionally make a refresh request
    $fitbit->setToken(getAccessTokenJsonFromMyDatabase());
} catch (\Exception $e) {
    print($e);
}
```

### Making a request

Inspect the FitbitPHPOAuth2 class to find the appropriate method. In this case, I want all activities on a date:
```php

try {
    $fitbit = new brulath\fitbit\FitbitPHPOAuth2(
        'your_client_id',
        'your_client_secret',
        'your_post_authorization_redirect_url',
        ['activity', 'heartrate', 'location', 'nutrition', 'profile', 'settings', 'sleep', 'social', 'weight'], // desired scopes
        true  // produce some debugging output in error_log
    );
    $fitbit->set_token(getAccessTokenJsonFromMyDatabase());
    
    $activities = $fitbit->getActivities('2016-02-20');
    storeAccessTokenIfItChanged($fitbit->getToken());
    print_r($activities);
} catch (\Exception $e) {
    print($e);
}
```

## License

The MIT License (MIT).
