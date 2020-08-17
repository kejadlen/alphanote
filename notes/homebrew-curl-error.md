# Homebrew cURL

`curl` sometimes fails with a cert error:

```sh
curl: (60) SSL certificate problem: certificate has expired
More details here: https://curl.haxx.se/docs/sslcerts.html
```

The relevant homebrew issue: [#7667][7667]

[7667]: https://github.com/Homebrew/brew/issues/7667

Though the comments imply that `CURL_SSL_BACKEND=secure-transport` should work
on 10.15, I had to use `HOMEBREW_FORCE_BREWED_CURL=1` instead.
