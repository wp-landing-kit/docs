# PHP API

- [Introduction](#introduction)
- [Usage](#usage)
    - [Inserting a new domain into the database](#inserting-a-new-domain-into-the-database)
    - [Getting an existing domain](#getting-an-existing-domain)
    - [Checking if a domain already exists in the database](#checking-if-a-domain-already-exists-in-the-database)
    - [Checking if an existing domain is active](#checking-if-an-existing-domain-is-active)
    - [Updating a domain](#updating-a-domain)
    - [Activating a domain](#activating-a-domain)
    - [Deactivating a domain](#deactivating-a-domain)
    - [Mapping the domain root](#mapping-the-domain-root)
    - [Controlling fallback behaviour on a domain](#controlling-fallback-behaviour-on-a-domain)
    - [Adding a URL to a domain](#adding-a-url-to-a-domain)
    - [Deleting a domain](#deleting-a-domain)

## Introduction

[WP Landing Kit](https://wplandingkit.com/) provides a simple PHP API to enable developers to programmatically work with domains. The API makes it
possible to:

1. Add new domain entries to the database.
1. Edit existing domain entries.
1. Delete domain entries.

When creating or editing a domain, the API provides control over:

1. Domain URL mappings (you can think of these as routes/endpoints)
    - Mappings can resolve to posts/pages/archives
    - Mappings can also redirect to both relative and absolute URLs
1. Domain settings
    - Protocol enforcement
    - Active status
    - Domain ownership (which WordPress user is the 'author' of the domain)

### Important!

It is important to keep in mind that the API affects the database and should not be used as part of a typical WordPress
page load. Instead, it is designed to be used at strategic points as a means of automating domain creation/manipulation.
e.g; You may choose to run this after a WooCommerce/Easy Digital Downloads transaction or perhaps after user
registration, etc.

## Usage

### Inserting a new domain into the database

You may create new domains using the `wplk_add_domain()` function. The function will return a `WPLK_Domain` object on
success or a `WP_Error` object when the domain is already in the database or some other failure occurs. Newly created
domains are inactive by default — that is, they are in a `draft` status and are not actively handling requests made to
the site. See [Activating a domain](#activating-a-domain).

```php
// Create an inactive domain with no mappings.
$domain = wplk_add_domain( 'mydomain.com' );

// Create an inactive domain with the domain root mapped to a post.
$post_id = 1234;
$domain = wplk_add_domain( 'mydomain.com', $post_id );

// Create an active domain with the domain root mapped to a post.
$post_id = 1234;
$domain = wplk_add_domain( 'mydomain.com', $post_id, true );

// Create an active domain with an owner (user ID), an enforced protocol, and with the root mapped to a post.
$post_id = 1234;
$user_id = 5678;
$domain = wplk_add_domain( 'mydomain.com', [
    'post_id' => $post_id,
    'owner_id' => $user_id,
    'active' => true,
    'enforced_protocol' => 'https',
] );
```

### Getting an existing domain

You may get an existing domain using the `wplk_get_domain()` function. The function will return a `WPLK_Domain` object
if the domain is found otherwise it will return `NULL`. You can pass this function one of the following:

1. The domain post ID.
1. The domain host name. e.g; `mydomain.com`.
1. A full URL containing the domain host name. e.g; `http://mydomain.com/some/path`.
    - In this case, the host name will be extracted from the URL. All other elements of the URL will be ignored.

**Note:** The most efficient lookup is achieved using the post ID so use that if/when possible.

```php
// Lookup using the domain post ID.
$domain = wplk_get_domain( 5 );

// Lookup using the domain host name:
$domain = wplk_get_domain( 'mydomain.com' );

// Lookup using a full URL containing the domain host name:
$domain = wplk_get_domain( 'http://mydomain.com/some/path' );
```

### Checking if a domain already exists in the database

You may check if a domain already exists by using the `wplk_domain_exists()` function. The function will return boolean
`TRUE` or `FALSE`. You can lookup the domain using any of the methods used in [Getting an existing domain](#getting-an-existing-domain).

```php
// Check using the domain post ID.
if( wplk_domain_exists( 5 ) ){
    // …
}

// Check using the domain host name.
if( wplk_domain_exists( 'mydomain.com' ) ){
    // …
}

// Check using a full URL containing the domain host name.
if( wplk_domain_exists( 'http://mydomain.com/some/path' ) ){
    // …
}
```

### Checking if an existing domain is active

You may check if an existing domain is active using the `wplk_domain_is_active()` function. The function can accept
either the domain ID, the domain host name, a full URL containing the host name, or a `WPLK_Domain` instance. The
function will return boolean `TRUE` or `FALSE` if the domain exists. IF the domain cannot be found, a `WP_Error` object
will be returned.

```php
// Check using the domain post ID.
if( wplk_domain_is_active( 5 ) ){
    // …
}

// Check using the domain host name.
if( wplk_domain_is_active( 'mydomain.com' ) ){
    // …
}

// Check using a full URL containing the domain host name.
if( wplk_domain_is_active( 'http://mydomain.com/some/path' ) ){
    // …
}

// Check using a WPLK_Domain instance.
$domain = wplk_get_domain( 'mydomain.com' );
if( $domain and wplk_domain_is_active( $domain ) ){
    // …
}
```

### Updating a domain

To save any changes made on a `WPLK_Domain` object to the database, you may pass the object to the `wplk_update_domain()`
function. The function will return the domain post ID on success or a `WP_Error` object on failure.

```php
$domain = wplk_get_domain( 'mydomain.com' );

// …make changes on the WPLK_Domain instance here…

$saved = wplk_update_domain( $domain );

if( is_wp_error( $saved ) ){
    // handle errors…
}
```

### Activating a domain

If a domain is eligible for activation, you may activate it using the `wplk_activate_domain()` function. The function
can accept either the domain ID, the domain host name, a full URL containing the host name, or a `WPLK_Domain` instance.
The function will return boolean `TRUE` on success or a `WP_Error` object where activation cannot be handled.

```php
// Activate using the domain post ID.
wplk_activate_domain( 5 );

// Activate using the domain host name.
wplk_activate_domain( 'mydomain.com' );

// Activate using a full URL containing the domain host name.
wplk_activate_domain( 'http://mydomain.com/some/path' );

// Activate using a WPLK_Domain instance.
$domain = wplk_get_domain( 'mydomain.com' );
if( $domain ){
    wplk_activate_domain( $domain );
}
```

### Deactivating a domain

You may deactivate a domain using the `wplk_deactivate_domain()` function. The function can accept either the domain ID,
the domain host name, a full URL containing the host name, or a `WPLK_Domain` instance. The function will return boolean
`TRUE` on success or a `WP_Error` object where activation cannot be handled.

```php
// Deactivate using the domain post ID.
wplk_deactivate_domain( 5 );

// Deactivate using the domain host name.
wplk_deactivate_domain( 'mydomain.com' );

// Deactivate using a full URL containing the domain host name.
wplk_deactivate_domain( 'http://mydomain.com/some/path' );

// Deactivate using a WPLK_Domain instance.
$domain = wplk_get_domain( 'mydomain.com' );
if( $domain ){
    wplk_deactivate_domain( $domain );
}
```

### Mapping the domain root

The domain root mapping is essential to the domain as it specifies what to load when a user visits the domain without
any URL path. e.g; `http://mydomain.com/`. Without the root mapping, a domain cannot be activated.

If the domain root should map to a post, you may specify the root mapping by passing the post ID to the
`wplk_add_domain()` function like so;

```php
wplk_add_domain( 'mydomain.com', $post_id );
```

For more examples of this shorthand approach, see [Inserting a new domain into the database](#inserting-a-new-domain-into-the-database).

Alternatively, you may choose to map the domain root to a taxonomy term archive, a post type archive, or even redirect
the domain root to a different URL entirely. To do so, you may chain method calls off the `root()` method which returns
a `WPLK_Mapping` object. Consider the following examples:

```php
$domain = wplk_add_domain( 'mydomain.com' );

// Map the root to a post using either a post ID or WP_Post object.
$domain->root()->maps_to_post( $post );

// Map the root to a taxonomy term archive using either a term ID or WP_Term object.
$support_pagination = true;
$domain->root()->maps_to_term_archive( $term, $support_pagination );

// Map the root to a post type archive using a post type slug.
$support_pagination = true;
$domain->root()->maps_to_post_type_archive( 'my_post_type_slug', $support_pagination );

// Redirect the root.
$domain->root()->redirects_to( 'http://somesite.com', '302' );
```

For more information about managing root mappings, see [WPLK_Domain > Working with the root mapping](wplk_domain.md#working-with-the-root-mapping).

### Controlling fallback behaviour on a domain

The fallback mapping on domain is used when an incoming request cannot be matched to any other mapping set up on a
domain. By default, the fallback behaviour of a domain will redirect to the domain root with a HTTP status of 302. You
may override the default behaviour in order to map the fallback to any post, post type archive, taxonomy term archive,
or to redirect to a URL of your choice. To do so, you may chain method calls off the `fallback()` method which returns
a `WPLK_Mapping` object. Consider the following examples:

```php
$domain = wplk_add_domain( 'mydomain.com' );

// Map the root to a post using either a post ID or WP_Post object.
$domain->fallback()->maps_to_post( $post );

// Map the root to a taxonomy term archive using either a term ID or WP_Term object.
$support_pagination = true;
$domain->fallback()->maps_to_term_archive( $term, $support_pagination );

// Map the root to a post type archive using a post type slug.
$support_pagination = true;
$domain->fallback()->maps_to_post_type_archive( 'my_post_type_slug', $support_pagination );

// Redirect the root.
$domain->fallback()->redirects_to( 'http://somesite.com', '302' );
```

### Adding mappings to a domain

If a domain needs more than just a root mapping, you may add any number of mappings to a domain using the `add_mapping()`
method on the `WPLK_Domain` instance. When invoked, the method creates a `WPLK_Mapping` object, adds it to the
`WPLK_Domain` object, and returns the mapping object for further configuration. Consider the following examples:

```php
$domain = wplk_add_domain( 'mydomain.com' );

// Map a path to a post using either a post ID or a WP_Post object.
$domain->add_mapping( 'some/path', $post );

// Map a path to a post using either a post ID or a WP_Post object using a chained method call.
$domain->add_mapping( 'some/path' )->maps_to_post( $post );

// Map a path to a taxonomy term archive using either a term ID or WP_Term object.
$support_pagination = true;
$domain->add_mapping( 'some/path' )->maps_to_term_archive( $term, $support_pagination );

// Map a path to a post type archive using the post type slug.
$support_pagination = true;
$domain->add_mapping( 'some/path' )->maps_to_post_type_archive( 'my_post_type_slug', $support_pagination );

// Perform multiple operations on the mapping object.
$mapping = $domain->add_mapping();
$mapping->set_url_path( 'some/path' );
$mapping->redirects_to( 'http://example.com' );
```

The `add_mapping()` method actually provides a number of alternative ways to create and configure mappings which
demonstrated in [WPLK_Mapping > Adding URL mappings](wplk_domain.md#adding-url-mappings)

### Adding a URL to a domain

todo

### Deleting a domain

todo