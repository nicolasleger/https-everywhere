# Contributing Code


* * *

# Contributing Rulesets

A note on terminology:

- `rule`: a specific regex rewrite that is applied for all matching `targets` within the same `ruleset`.  There may be many `rules` within any given `ruleset`.
- `target`: a FQDN which may include a wildcard specified by `*.` on the left side, which `rules` are applied to.  There may be many `targets` within any given `ruleset`.
- `test`: a URL for which a request is made to ensure that the rewrite is working properly.  There may be many `tests` within any givin `ruleset`.
- `ruleset`: a scope in which `rules`, `targets`, and `tests` are contained.  `rulesets` usually are named after the entity which controls the group of `targets` contained in it.  There is one `ruleset` per file within the s`rc/chrome/content/rules` directory.

## General Info

Thanks for your interest in contributing to the HTTPS Everywhere `rulesets`!  There's just a few things you should know before jumping in.

HTTPS Everywhere includes tens of thousands of `rulesets`.  Any one of these sites can change their HTTPS configuration at any time, so keeping HTTPS Everywhere usable is a task that requires constant maintenance.  At the same time, HTTPS deployment on the web is becoming more and more widespread, thanks to projects like [Let's Encrypt](https://letsencrypt.org/).  This is a very good thing, as it means the web is becoming a safer place!  However, with each new `ruleset` that HTTPS Everywhere includes comes with an increase in both download size upon install and memory usage at runtime.  Rather than adding new `rulesets`, we encourage potential contributors to look for broken `rulesets` and try to fix them first.

## New Rulesets

If you want to create new `rulesets` to submit to us, we expect them to be in the s`rc/chrome/content/rules` directory. That directory also contains a useful script, `make-trivial-rule`, to create a simple `ruleset` for a specified domain. There is also a script called `utils/trivial-validate.py`, to check all the pending `rulesets` for several common errors and oversights. For example, if you wanted to make a `ruleset` for the `example.com` domain, you could run:
```
cd src/chrome/content/rules
bash ./make-trivial-rule example.com
```
This would create `Example.com.xml`, which you could then take a look at and edit based on your knowledge of any specific URLs at `example.com` that do or don't work in HTTPS. Please have a look at our [Ruleset Style Guide](https://github.com/EFForg/https-everywhere/blob/master/ruleset-style.md) where you can find useful tips about finding more subdomains. Our goal is to have as many subdomains covered as we can find.

## Minimum Requirements for a Ruleset PR

Try to enumerate as many domains as posible...
Your commit may be squashed and merged...
Indent exclusions and tests to the target....
Try fixing rulesets first instead of contributing new ones...

## Ruleset Style Guide

TODO: This needs to be merged with the styleguide in #7707 in some sensible way

Goal: rules should be written in a way that is consistent, easy for humans to read and debug, reduces the chance of errors, and makes testing easy.

To that end, here are some style guidelines for writing or modifying rulesets. They are intended to help and simplify in places where choices are ambiguous, but like all guidelines they can be broken if the circumstances require it.

Avoid using the left-wildcard (`<target host='*.example.com' />`) unless you intend to rewrite all or nearly all subdomains.  Many rules today specify a left-wildcard target, but the rewrite rules only rewrite an explicit list of hostnames.

Instead, prefer listing explicit target hosts and a single rewrite from `"^http:"` to `"^https:"`. This saves you time as a ruleset author because each explicit target host automatically creates an implicit test URL, reducing the need to add your own test URLs. These also make it easier for someone reading the ruleset to figure out which subdomains are covered.

If you know all subdomains of a given domain support HTTPS, go ahead and use a left-wildcard, along with a plain rewrite from `"^http:"` to `"^https:"`. Make sure to add a bunch of test URLs for the more important subdomains. If you're not sure what subdomains might exist, you can install the `Sublist3r` tool:

    git clone https://github.com/aboul3la/Sublist3r.git
    cd Sublist3r
    sudo pip install -r requirements.txt # or use virtualenv...

Then you can to enumerate the list of subdomains:

    python sublist3r.py -d example.com -e Baidu,Yahoo,Google,Bing,Ask,Netcraft,Virustotal,SSL

Alternatively, you can iteratively use Google queries and enumerate the list of results like such:

1. site:*.eff.org
2. site:*.eff.org -site:www.eff.org
3. site:*.eff.org -site:www.eff.org -site:ssd.eff.org

... and so on.

If there are a handful of tricky subdomains, but most subdomains can handle the plain rewrite from `"^http:"` to `"^https:"`, specify the rules for the tricky subdomains first, and then then plain rule last. Earlier rules will take precedence, and processing stops at the first matching rule. There may be a tiny performance hit for processing exception cases earlier in the ruleset and the common case last, but in most cases the performance issue is trumped by readability.

Avoid regexes with long strings of subdomains, e.g. `<rule from="^http://(foo|bar|baz|bananas).example.com" />`. These are hard to read and maintain, and are usually better expressed with a longer list of target hosts, plus a plain rewrite from `"^http:"` to `"^https:"`.

Prefer dashes over underscores in filenames. Dashes are easier to type.

Use tabs and double quotes (`"`, not `'`).

When matching an arbitrary DNS label (a single component of a hostname), prefer `([\w-]+)` for a single label (i.e. www), or `([\w.-]+)` for multiple labels (i.e. www.beta). Avoid more visually complicated options like `([^/:@\.]+\.)?`.

For `securecookie` tags, if you know that all cookies on the included targets can be secured (which in particular means that the cookies are not used by any of its non-securable subdomains), use the trivial

```xml
<securecookie host=".+" name=".+" />
```

where we prefer `.+` over `.*` and `.`. They are functionally equivalent, but it's nice to be consistent.

Avoid the negative lookahead operator `?!`. This is almost always better expressed using positive rule tags and negative exclusion tags. Some rulesets have exclusion tags that contain negative lookahead operators, which is very confusing.

Prefer capturing groups `(www\.)?` over non-capturing `(?:www\.)?`. The non-capturing form adds extra line noise that makes rules harder to read. Generally you can achieve the same effect by choosing a correspondingly higher index for your replacement group to account for the groups you don't care about.

Avoid snapping redirects. For instance, if https://foo.fm serves HTTPS correctly, but redirects to https://foo.com, it's tempting to rewrite foo.fm to foo.com, to save users the latency of the redirect. However, such rulesets are less obviously correct and require more scrutiny. And the redirect can go out of date and cause problems. HTTPS Everywhere rulesets should change requests the minimum amount necessary to ensure a secure connection.

Here is an example ruleset pre-style guidelines:

```xml
<ruleset name="WHATWG.org">
  <target host='whatwg.org' />
  <target host="*.whatwg.org" />

  <rule from="^http://((?:developers|html-differences|images|resources|\w+\.spec|wiki|www)\.)?whatwg\.org/"
    to="https://$1whatwg.org/" />
</ruleset>
```

Here is how you could rewrite it according to these style guidelines, including
test URLs:

```xml
<ruleset name="WHATWG.org">
	<target host="whatwg.org" />
	<target host="developers.whatwg.org" />
	<target host="html-differences.whatwg.org" />
	<target host="images.whatwg.org" />
	<target host="resources.whatwg.org" />
	<target host="*.spec.whatwg.org" />
	<target host="wiki.whatwg.org" />
	<target host="www.whatwg.org" />

	<test url="http://html.spec.whatwg.org/" />
	<test url="http://fetch.spec.whatwg.org/" />
	<test url="http://xhr.spec.whatwg.org/" />
	<test url="http://dom.spec.whatwg.org/" />

	<rule from="^http:"
		to="https:" />
</ruleset>
```

## Removal of Rules

### Regular Rules

It should be considered a sufficient condition for removal if a contributor can demonstrate that the TLS configuration for either a specific `target` or a ruleset altogether is unstable and/or breaking, or will be unstable and/or breaking in the near future.  It is, of course, preferrable that the `ruleset` be fixed rather than removed.

### HSTS Preloaded Rules

In `utils` we have a tool called `hsts-prune` which removes `targets` from rulesets if they are already contained in the [HSTS preload](https://hstspreload.org/) list for browsers that we support.  To be explicit, the script is an implementation of the following policy:

> Let `included domain` denote either a `target`, or a parent of a `target`.  Let s`upported browsers` include the ESR, Dev, and Stable releases of Firefox, and the Stable release of Chromium.  If `included domain` is a parent of the `target`, the `included domain` must be present in the HSTS preload list for all s`upported browsers` with the relevant flag which denotes inclusion of subdomains set to `true`.  If `included domain` is the `target` itself, it must be included the HSTS preload list for all s`upported browsers`.  Additionally, if the http endpoint of the `target` exists, it must issue a 3XX redirect to the https endpoint for that target.  Additionally, the https endpoint for the `target` must deliver a `Strict-Transport-Security` header with the following directives present:
>
> - `max-age` >= 10886400
> - `includeSubDomains`
> - `preload`
>
> If all the above conditions are met, a contributor may remove the `target` from the HTTPS Everywhere rulesets.  If all targets are removed for a ruleset, the contributor is advised to remove the ruleset file itself.  The ruleset `rule` and `test` tags may need to be modified in order to pass the ruleset coverage test.

Every new pull request automatically has the `hsts-prune` utility applied to it as part of the continual integration process.  If a new PR introduces a `target` which is preloaded, it will fail the CI test suite.  See:

- `.travis.yml`
- `test/travis.sh`

* * *

# Contributing Translations

HTTPS Everywhere translations are handled through Transifex.  The easiest way to help with translations is to [create a Transifex account](https://www.transifex.com/signup/) if you don't already have one.  Then log into your account and click "Explore", then search for "Tor Project", and click on The Tor Project.  Then choose the language you plan to translate into, click on the name of that language, and then click "Join team" and "Go" to accept joining the translation team for your language.

Then, in the Tor Project resources list, find and click the link for the file

    HTTPS Everywhere - https-everywhere.dtd

and choose "Translate now" to enter the translation interface.

* * *

A more detailed guide about the syntax can be found at [EFF.org](https://www.eff.org/https-everywhere/rulesets).
For questions see our [FAQ](https://www.eff.org/https-everywhere/faq) page.
