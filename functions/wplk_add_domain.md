```php
wplk_add_domain( string $domain_name, int|array $config = [], boolean $active = false )
```

## Description

The function inserts new domains into the database.

## Parameters

### $domain_name

&nbsp;&nbsp; (string) (Required) The domain host name or a full URL containing the host name.

### $config

&nbsp;&nbsp; (int|array) (Optional) A post ID or an array of settings.

### $active

&nbsp;&nbsp; (boolean) (Optional) Whether or not the domain is currently active (published).

## Return

Returns `WPLK_Domain` object on success or a `WP_Error` object on failure.

## Examples

```php
$home_post_id = 1234;
$domain = wplk_add_domain( 'mydomain.com', $home_post_id );

if( is_wp_error( $domain ) ){
    // do something with $domain->get_error_message()
}
```

## Notes