# Python IndieWeb Utilities

*NOTE: This library is not yet ready for release or use.*

This Python library contains utilities to aid the implementation of various IndieWeb specifications and functionalities.

## Feature Set

This package provides functions that cater to the following needs:

- Generating reply context for a given page.
- Finding the original version of a post per the [Original Post Discovery](https://indieweb.org/original-post-discovery#Algorithm) specification.
- Finding the post type per the [Post Type Discovery](https://ptd.spec.indieweb.org/) W3C note.
- Finding the [webmention endpoint on a page](https://webmention.net/draft/#sender-discovers-receiver-webmention-endpoint), if one is provided.
- Canonicalizing a URL.
- Discovering the author of a post per the [Authorship](https://indieweb.org/authorship-spec) Specification.
- Handling the response from an IndieAuth callback request.

## Why Use indieweb-utils?

If any of the above use cases resonate with you, this library may be helpful. Please note this library does not fully implement all IndieWeb specifications. Rather, it provides a set of building blocks that you can use to speed up your development of IndieWeb applications.

The following applications will benefit from at least one of the functions provided in this library:

- Micropub server.
- Microsub server.
- Webmention sender.
- Any application that needs to canonicalize a URL.
- An application implementing IndieAuth.

## Installation

To install this package, run the following command:

    pip install indieweb-utils

You can import the package using the following line of code:

    import indieweb_utils

## Getting Started

Use the links below to guide you to the relevant documentation for your needs.

- [Find a post type.](#find-a-post-type)
- [Discover a webmention endpoint.](#discover-a-webmention-endpoint)
- [Canonicalize a URL.](#canonicalize-a-url)
- [Discover an article author.](#discover-an-article-author)
- [Handle an IndieAuth callback request.](#handle-an-indieauth-callback-request)
- [Generate reply context.](#generate-reply-context)
- [Find the original version of a post.](#find-the-original-version-of-a-post)

### Find a Post Type

To find the post type associated with a web page, you can use the `get_post_type` function.

The `get_post_type` function function uses the following syntax:

    indieweb_utils.post_type_discovery.get_post_type(url, custom_properties=[])

Here are the arguments you can use:

- `url`: The URL of the web page to discover the post type of.
- `custom_properties`: A list of custom properties to use when discovering the post type. This list must contain tuples.

The function returns a single string with the post type of the specified web page.

See the Post Type Discovery specification for a full list of post types.

#### Custom Properties

The structure of the custom properties tuple is:

(attribute_to_look_for, value_to_return)

An example custom property value is:

    custom_properties = [
        ("poke-of", "poke")
    ]

This function would look for a poke-of attribute on a web page and return the "poke" value.

By default, this function contains all of the attributes in the Post Type Discovery mechanism.

Custom properties are added to the end of the post type discovery list, just before the "article" property. All specification property types will be checked before your custom attribute.

Here is an example of the `get_post_type` function in use:

    import indieauth_helpers

    url = "https://jamesg.blog/2021/12/06/advent-of-bloggers-6/"

    post_type = indieweb_utils.post_type_discovery.discover_endpoint(url)

    print(post_type)

This code returns the following string:

    article

### Discover a Webmention Endpoint

Webmention endpoint discovery is useful if you want to know if you can send webmentions to a site or if you want to send a webmention to a site.

You can discover if a URL has an associated webmention endpoint using this code:

    import indieauth_helpers

    url = "https://jamesg.blog/2021/12/06/advent-of-bloggers-6/"

    url, message = indieauth_helpers.webmention.discover_webmention_endpoint(url)

    print(url)

If successful, this function will return the URL of the webmention endpoint associated with a resource. The message value will be a blank string in this acse.

If a webmention endpoint could not be found, URL will be equal to None. A string message value will be provided that you can use for debugging or present to a user.

### Canonicalize a URL

Canonicalization turns a relative URL into a complete URL.

To canonicalize a URL, use this function:

    url = indieweb_utils.canonicalize.canonicalize_url(old_url, domain)

This function requires two arguments.

- `url`: The URL to canonicalize.
- `domain`: The domain to use during canonicalization.

This function will return a URL with a protocol, host, and path.

The domain of the resource is needed so that it can be added to the URL during canonicalization if the URL is relative.

A complete URL returned by this function will look like this:

    https://indieweb.org/POSSE

### Discover an Article Author

You can discover the original author of an article as per the Authorship Specification.

To do so, use this function:

    post_type = indieauth_helpers.authorship_discovery.discover_author(url)

Here are the arguments you can use:

- `url`: The URL of the web page whose author you want to discover.
- `page_contents`: The unmodified HTML of a web page whose author you want to discover.

The page_contents argument is optional.

If no page_contents argument is specified, the URL you stated will be retrieved and authorship inference will begin.

If you specify a page_contents value, the HTML you parsed will be used for authorship discovery. This will save on a HTML request if you have already retrieved the HTML for another reason (for example, if you need to retreive other values in the page HTML). You still need to specify a URL even if you specify a page_contents value.

The discover_author function can return one of two values:

- An author name (i.e. "James").
- The h-card of an author.

These are the two outputs defined in the authorship inference algorithm. Your program should be able to handle both of these outputs.

Here is an example of the `discover_author` function in action:

    import indieauth_helpers

    url = "https://aaronparecki.com/2021/12/07/8/drone"

    post_type = indieauth_helpers.authorship_discovery.discover_author(url)

    print(post_type)

This code returns the following h-card:

    {
        'type': ['h-card'],
        'properties': {
            'url': ['https://aaronparecki.com/'],
            'name': ['Aaron Parecki'],
            'photo': ['https://aaronparecki.com/images/profile.jpg']
        },
        'value': 'https://aaronparecki.com/'
    }

### Handle an IndieAuth Callback Request

The last stage of the IndieAuth authentication flow for a client is to verify a callback response and exchange the provided code with a token.

This function implements a callback handler to verify the response frmo an authorization server and redeem a token.

To use this function, you need to pass in the following arguments:

    message, response = indieweb_utils.auth.indieauth_callback_handler(
        code, # The code that was returned by the IndieAuth server
        state, # The state that was returned by the IndieAuth server
        token_endpoint, # The token endpoint to which the callback POST request should be sent
        code_verifier, # The code verifier that was used to generate the code
        session_state, # The session state that was generated by the client
        me, # The URL of the user's profile
        callback_url, # The URL to which the user should be redirected if authentication is successful
        client_id, # The URL of the page that identifies the client
        required_scopes # The scopes that the client needs
    )

This function verifies that an authorization server has returned a valid response and redeems a token.

You can leave the "me" value equal to None if any URL should be able to access your service. Otherwise, set "me" to the URL of the profile that should be able to access your service. Setting a me value other than None may be useful if you are building personal services that nobody else should be able to access.

If successful, this function will return a None value for "message" and the JSON object returned by an IndieAuth authorization server. This JSON object should look like this:

    {
        "me": "https://jamesg.blog/",
        "access_token": "ACCESS_TOKEN",
        "scope": "SCOPE_LIST"
    }

The function will return a message and a None value for response if there was an error. The message value tells you what went wrong during the token verification process.

### Generate Reply Context

To generate reply context for a given page, use the following function:

    reply_context = indieweb_utils.context.generate_reply_context(url)

This function will return a dictionary with the following keys:

    {
        "p-name": "The title of the page",
        "post_body": "The body of the page",
        "author_name": "The name of the author",
        "author_url": "The URL of the author",
        "author_image": "The URL of the author's photo",
    }

A None value will be specified if any of the above attributes cannot be found.

### Find the Original Version of a Post

To find the original version of a post, use this code:

    original_post = indieweb_utils.original_post_discovery.discover_original_post(url)

This function will return the URL of the original version of a post, if one is found. Otherwise, None will be returned.

## License

This project is licensed under the [MIT license](LICENSE).

## Dependencies

This project uses the following dependencies:

- BeautifulSoup4 for HTML parsing
- mf2py for microformats parsing
- requests for making HTTP requests

## Contributing

This project welcomes contributions from anyone who wants to improve the library.

Feel free to create an issue or pull request on GitHub and your contribution will be reviewed.

## Contributors

- capjamesg
