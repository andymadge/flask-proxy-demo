flask-proxy-demo
================

Kind of intended to be an example of what NOT to do when running a Flask app behind Heroku or another proxy.

See http://esd.io/blog/flask-apps-heroku-real-ip-spoofing.html


## Additional Notes

The X-Forwarded-For header should not be trusted because it's very easy to spoof.  ONLY the last (right-most) entry can be trusted because that is set by the receiving server (yours).

If you have reverse proxies in front of your server and trust them, then you can move further back up the list.  The rightmost non-trusted IP will be the real client IP.

Werkzeug >= 0.9 includes patched ProxyFix to prevent this spoofing by using the trusted proxy technique.

The fix assumes there is only a single TRUSTED reverse proxy in use.  If you have multiple reverse proxies and **YOU TRUST THEM ALL** you can tell it how many to trust by changing:

    app.wsgi_app = ProxyFix(app.wsgi_app)
	
to:

    app.wsgi_app = ProxyFix(app.wsgi_app, num_proxies=2)

However, as noted in http://stackoverflow.com/a/22936947/46433 you should be wary of the 'counting proxies' approach - it only works if you trust the environment.  If you move the code elsewhere it may then be trusting non-trustworthy proxies.

See http://stackoverflow.com/a/22251171/46433 for a more explicit approach which specifies the proxies to trust.


### Branches in this fork:

`master` - Werkzeug 0.8.3 with ProxyFix (as upstream) + updated readme.

`master-noProxyfix` - Werkzeug 0.8.3 WITHOUT ProxyFix.

`current` - Werkzeug 0.12.1 with ProxyFix.

`current-noProxyfix` - Werkzeug 0.12.1 WITHOUT ProxyFix.


### Tests with the ProxyFix enabled

Push each branch to Heroku then test with and without spoofing:

- `5.5.5.5` substituted for real IP
- `https://example.herokuapp.com/` substituted for Heroku app URL


#### master Branch Test - Spoofable

Deploy with `git push heroku master --force`

    $ curl https://example.herokuapp.com/
    Your IP address is 5.5.5.5
	
    $ curl -H "X-Forwarded-For: 1.2.3.4" https://example.herokuapp.com/
    Your IP address is 1.2.3.4
	
	$ curl -H "X-Forwarded-For: 3.3.3.3, 1.2.3.4, 4.4.4.4" https://example.herokuapp.com/
	Your IP address is 3.3.3.3
	

#### current Branch Test - NOT Spoofable

Deploy with `git push heroku current:master --force`

    $ curl https://example.herokuapp.com/
    Your IP address is 5.5.5.5
	
    $ curl -H "X-Forwarded-For: 1.2.3.4" https://example.herokuapp.com/
    Your IP address is 5.5.5.5
	
	$ curl -H "X-Forwarded-For: 3.3.3.3, 1.2.3.4, 4.4.4.4" https://example.herokuapp.com/
	Your IP address is 5.5.5.5
