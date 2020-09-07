# PHP API

- [Introduction](#introduction)
- [Functions](#functions)
- [Objects](#objects)
  - [WPLK_Domain](#wplk_domain)
    - [Creating a new domain](#creating-a-new-domain)
    - [Retrieving an existing domain](#retrieving-an-existing-domain)
    - [Managing URL mappings](#managing-url-mappings)
  - [WPLK_Mapping](#wplk_mapping)

## Introduction

WP Landing Kit provides a simple PHP API to enable developers to programmatically work with domains. The API makes it
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

## Functions

todo

## Objects

todo

### WPLK_Domain

The `WPLK_Domain` object is the backbone of domain management and is the underlying component powering all of the API
functions. The class can be used to:

1. Retrieve existing domain instances from the database and;
1. Creation of new instances for configuration and insertion into the database.

#### Creating a new domain

You can create a new domain object by instantiating the object with a variety of contructor args.

```php
// Simply pass the domain host name on instantiation.
$domain = new WPLK_Domain( 'mydomain.com' );

// Note, you can also pass a full URL — the object will extract the `mydomain.com` host name in this case.
$domain = new WPLK_Domain( 'http://mydomain.com/some/url' );

// Passing no args to the constructor requires the use of the set_host() method to set the actual domain host name. If
// no host name is set, subsequent calls to $domain->save() will return a WP_Error object and the domain will not be
// saved to the database.
$domain = new WPLK_Domain;
$domain->set_host( 'mydomain.com' );

// It is also possible to use an array when instantiating a new domain object.
$domain = new WPLK_Domain( [
    'host' => 'mydomain.com',
    'is_active' => true, // (optional)
    'enforced_protocol' => 'https', // (optional)
] );
```

Be mindful that instantiating a domain object does not add it to the database. It merely creates an object for you to
further configure and save. Until a domain is explicitly activated and saved to the database, the domain will not handle
requests made to the site.

The following example is a more complete demonstration of how to create and activate a domain. In this example, we are:

1. Creating a new domain object;
1. Setting the post/page to show as the domain's root/home;
1. Setting the active flag on the domain;
1. Saving the domain to the database;
1. Checking for any errors on save.

```php
$domain = new WPLK_Domain( 'mydomain.com' );
$domain->root()->maps_to_post( $post_id );
$domain->activate();

$saved = $domain->save();

if( is_wp_error( $saved ) ){
    // …do something…
}
```

#### Retrieving an existing domain

You can retrieve an instance of a domain that already stored in the database using the `WPLK_Domain::get_instance()`
method. The method returns a `WPLK_Domain` instance when it can find a post or `NULL` when it cannot.

The method accepts either a post ID or a domain host name. e.g;

```php
// Using a post ID.
$domain = WPLK_Domain::get_instance( $post_id );

// Using a host name.
$domain = WPLK_Domain::get_instance( 'mydomain.com' );

// Using a full URL (the host name will be extraced from the URL).
$domain = WPLK_Domain::get_instance( 'http://mydomain.com/some/url' );
```

Be mindful that the most performant option is to use the post ID. Using the host name resorts to searching for the
underlying domain post by title.

#### Managing URL mappings

todo

### WPLK_Mapping

todo