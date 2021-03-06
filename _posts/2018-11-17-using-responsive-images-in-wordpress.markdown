---
layout: post
title:  "Using Responsive Images in WordPress"
date:   2018-11-17 15:38:01 -0600
---

Images generally account for a large amount data sent when serving a website. In this post I'll share some ways you can save users some bytes by serving images that fit the size of the device. There are a couple of tips you should know when using responsive images in WordPress.

## Srcset
In 2015 WordPress rolled out some handy [functions](https://make.wordpress.org/core/2015/11/10/responsive-images-in-wordpress-4-4/) for making images responsive in WordPress. This is done by automatically generating the `srcset` attribute and multiple sizes for images loaded into the media library.

If you have not heard of `srcset`, let me break it down. Let's say we have cat.jpg and some sizes of that image already made for us. So the folder structure will look like this.

```
- my-project
  - index.html
  - cat.jpg
  - cat-small.jpg
  - cat-medium.jpg
  - cat-large.jpg
```

Then inside index.html we have an image tag.

``` html
<img
  src="/cat.jpg"
  srcset="/cat-small.jpg 500w, /cat-medium.jpg 940w, /cat-large.jpg 1200w"
  alt="cat"
>
```

When a modern browser encounters this img tag, it will compare the srcset against the size of the window. Notice how each image path in the `srcset` has a number after the path. This number tells the browser at which window size the image should be used. In the example above, cat-small.jpg will be used as the image src until the browser window is 500px wide. We include the `src` attribute as a fallback.

So, why does this matter? The beauty of `srcset` is that we can serve the best image size for a given device. The browser will only make a request for the image it needs. As developers, we provide the browser some options and it picks the best fit. Using this method, we can ensure retina screens will have high quality images without causing mobile users to run out of data after downloading one giant retina image they did not need to have a good mobile experience.

You can do a lot more with `srcset` but that's the basic gist.

By default, images used in `the_content` area of pages and posts will have a `srcset` generated by WordPress. What about when we want to use the `srcset` for images outside the editor. Let's look at how we could use this to display a post thumbnail in the loop.

``` php
<?php
  $post_thumbnail_id     = get_post_thumbnail_id(); // Returns the attachment id
  $post_thumbnail_srcset = wp_get_attachment_image_srcset( $post_thumbnail_id, 'medium' );
  // @see https://developer.wordpress.org/reference/functions/wp_get_attachment_image_srcset/
  // The first argument is the attachment id and the second is the lowest size that
  // will be included in the the srcset.
?>

<img
  src="<?php echo esc_url( get_post_thumbnail_url() ); ?>"
  srcset="<?php echo esc_attr( $post_thumbnail_srcset ); ?>"
  alt="My post thumbnail"
>
```

Now this can also be achieved by using `the_post_thumbnail()` but knowing how to get an image's srcset can be useful for other situations, such as when working with [Advanced Custom Fields](https://www.advancedcustomfields.com/).

``` php
<?php
  $image        = get_field( 'my_image' );
  $image_srcset = wp_get_attachment_image_srcset( $image['ID'], 'medium' );
?>

<img
  src="<?php echo esc_url( get_post_thumbnail_url() ); ?>"
  srcset="<?php echo esc_attr( $image_srcset ); ?>"
  alt="My custom image"
>
```

### Some notes about using wp_get_attachment_image_srcset()

Under the hood `wp_get_attachment_image_srcset` uses `wp_calculate_image_srcset`. There a couple of quirks you should know about when using WordPress' helper functions.

`wp_calculate_image_srcset` will only return images that match the ratio of the size provided. If 'medium' is passed, the function will return all the possible sizes above medium that have the same image height to width ratio in pixels. This can be tricky if you are working with any custom registered sizes as they may not have any other available sizes unless you register them.

If you register a custom image size that crops the image, consider adding other sizes with the same ratio. Otherwise you may get unexpected results from `wp_get_attachment_image_srcset` when working with custom sizes.

``` php
// Both sizes have the same ratio
add_image_size( 'sub-page', 1300, 400, true );
add_image_size( 'sub-page-small' , 650,200,true );
```

On that note, `wp_get_attachment_image_srcset` will fail if the image requested has less than two sizes in the media library. Using the example above, if I uploaded an image that is 1000px x 400px WordPress cannot make the 'sub-page' size because the image is not large enough.

If you start using this method to create full page images or banners, note that by default `wp_get_attachment_image_srcset` will not return an image larger than 1600px.

From wp-includes/media.php:
``` php
/**
  * Filters the maximum image width to be included in a 'srcset' attribute.
  *
  * @since 4.4.0
  *
  * @param int   $max_width  The maximum image width to be included in the 'srcset'. Default '1600'.
  * @param array $size_array Array of width and height values in pixels (in that order).
  */
$max_srcset_image_width = apply_filters( 'max_srcset_image_width', 1600, $size_array );
```

This is easy to remedy by adding a filter.
``` php
add_filter( 'max_srcset_image_width', 'max_srcset_image_width', 10 , 2 );

function max_srcset_image_width() {
	return 1930; // Desired max width in pixels.
}
```

Hopefully this helps you get started with responsive images in WordPress. if you want to learn more about responsive image, I've provided some helpful links. In an upcoming post I'll go over how we can combine this with lazyloading.

[W3C](http://usecases.responsiveimages.org/)

[Responsive Images - The srcset and sizes attributes](https://bitsofco.de/the-srcset-and-sizes-attributes/)
