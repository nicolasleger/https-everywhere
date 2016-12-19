# Contributing Code



# Contributing Rulesets

## General Info

If you want to create new rules to submit to us, we expect them to be in the `src/chrome/content/rules` directory. That directory also contains a useful script, `make-trivial-rule`, to create a simple rule for a specified domain. There is also a script called `utils/trivial-validate.py`, to check all the pending rules for several common errors and oversights. For example, if you wanted to make a rule for the `example.com` domain, you could run:
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

## Removal of Rules

### Regular Rules

It should be considered a sufficient condition for removal if a contributor can demonstrate that the TLS configuration for either a specific `target` or a ruleset altogether is unstable and/or breaking, or will be unstable and/or breaking in the near future.

### HSTS Preloaded Rules

In `utils` we have a tool called `hsts-prune` which removes `target`s from rulesets if they are already contained in the [HSTS preload](https://hstspreload.org/) list for browsers that we support.  To be explicit, the script is an implementation of the following policy:

Let `included domain` denote either a `target`, or a parent of a `target`.  Let `supported browsers` include the ESR, Dev, and Stable releases of Firefox, and the Stable release of Chromium.  If `included domain` is a parent of the `target`, the `included domain` must be present in the HSTS preload list for all `supported browsers` with the relevant flag which denotes inclusion of subdomains set to `true`.  If `included domain` is the `target` itself, it must be included the HSTS preload list for all `supported browsers`.  Additionally, if the http endpoint of the `target` exists, it must issue a 3XX redirect to the https endpoint for that target.  Additionally, the https endpoint for the `target` must deliver a `Strict-Transport-Security` header with the following directives present:

- `max-age` >= 10886400
- `includeSubDomains`
- `preload`

If all the above conditions are met, a contributor may remove the `target` from the HTTPS Everywhere rulesets.  If all targets are removed for a ruleset, the contributor is advised to remove the ruleset file itself.  The ruleset `rule` and `test` tags may need to be modified in order to pass the ruleset coverage test.

# Contributing Translations

HTTPS Everywhere translations are handled through Transifex.  The easiest way to help with translations is to [create a Transifex account](https://www.transifex.com/signup/) if you don't already have one.  Then log into your account and click "Explore", then search for "Tor Project", and click on The Tor Project.  Then choose the language you plan to translate into, click on the name of that language, and then click "Join team" and "Go" to accept joining the translation team for your language.

Then, in the Tor Project resources list, find and click the link for the file

    HTTPS Everywhere - https-everywhere.dtd

and choose "Translate now" to enter the translation interface.

* * *

A more detailed guide about the syntax can be found at [EFF.org](https://www.eff.org/https-everywhere/rulesets).
For questions see our [FAQ](https://www.eff.org/https-everywhere/faq) page.
