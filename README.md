LiipSearchBundle
================

This bundle allows using the Google XML API for searching content contained
in the site(s) that you have configured in a [Google site search](http://www.google.com/sitesearch/) service.

Perhaps other search services will be added to this bundle in the future (solr, etc).

Introduction
------------
This search bundle allows you to add search to your site.  It uses the Google Websearch
XML API as a backend service.
Provided for you are:

* A service which passes submitted queries on to google, and returns HTML results

* A service which provides paging for the search results

* A controller service which can be used to render a search box from a template


Configuration
-------------
These parameters can be configured in your config.yml:

* liip_search.search_route
  * default value: 'search'
  * This is the name of the route that will handle submitted search requests

* liip_search.restrict_to_site
  * default value: ''
  * example value: 'www.example.com'
  * With the default, empty value, all domains configured in the sitesearch account will be searched
  * You may specify a domain here to restrict the search to a single domain if you have configured several domains to search in your sitesearch account

* liip_search.restrict_by_language
  * default value: false
  * Change this to true if you want to restrict the search to results that Google thinks are in the language specified by the session locale

* liip_search.translation_domain
  * default value: 'liip_search_bundle_search'
  * Provides the name of the translation file to use.

* liip_search.results_per_page
  * default value: 10
  * How many search results to display per page

* liip_search.pager_max_extremity_items
  * default value: 2
  * How many page links to always show at the beginning and end of the search results
  * For example, with a value of 2, this always shows page 1, 2, n-1, n, where n is the last page of results.  It should do the right thing, e.g. it will NOT show something like 1,2,1,2 when there are only 2 pages of results.

* liip_search.pager_max_adjoining_items
  * default value: 2
  * How many page links to always show before and after the current page
  * For example, with a value of 2, on page 6 of the results, it would show <extremity pages> ... 4 5 *6* 7 8 ... <extremity pages>

* liip_search.query_param_name: q
  * default value: 'q'
  * The key string used for submitting the search term (e.g. /search?*q*=software)

* liip_search.page_param_name
  * default value: 'p'
  * The key string used for submitting the page number (e.g. /search?q=software&*p*=3)

Usage
-----
Include the bundle in your app/autoload.php and app/Kernel.php.

You can include the default search box by rendering the showSearchBox action of the default search controller:

    {% render 'liip_search_default_controller:showSearchBoxAction' with {'field_id':'query', 'query':'last_query'} %}

The parameters you must pass are:

* field_id - The ID of the html text field for the search. This parameter allows you to have more than one search box in a single page
* query - [optional] Allows you to specify the last searched term with which the search input field will be populated


Create a route for the search action.  You can either define your own action, or
use the liip_google_search:search action as the controller for the route.

If you define you own action, you'll need to provide the query and page parameters when
rendering the liip_google_search search action from the twig template.
Your custom search action method might look like this:

    use Liip\SearchBundle\Helper\SearchParams;
    ...
    public function searchAction()
    {
        return $this->render('MyBundle:Search:search.html.twig',
                array(
                    'title' => 'Search'
                    'query' => SearchParams::requestedQuery($request, $queryParamName),
                    'page'  => SearchParams::requestedPage($request, $pageParamName),
                ));
    }

Where MyBundle:Search:search.html.twig renders the liip_google_search search action:

    {% render "liip_google_search:searchAction" with {'query': query, 'page': page} %}

When rendering from a template like this, the query and page parameters must be provided.
When rendered from a template, a subrequest is used, and liip_google_search:searchAction
will not have access to the original Request object, and so cannot read the query and
page parameters from the Request.


If, on the other hand, you choose to use the liip_google_search:search action, your route
will look something like this:

    search:
        pattern: /search
        defaults: { _controller: liip_google_search:search }

If you're doing this, you'll want to override the templates so that you can include your
site-specific layout.

Overriding the templates
------------------------

The templates used by the bundle can be overridden by the normal Symfony2 mechanism to replace predefined
templates.

Your version of the templates must go into app/Resources.

See http://symfony.com/doc/2.0/book/templating.html#overriding-bundle-templates

Overriding the translations
---------------------------

The translations used by the bundle can be overridden by creating an XLIFF file with the correct translations
key and then setting the liip_search.translation_domain to the name of your translation file.


TODO
----
### for the google search service
* add support for Refinements
* add support for Synonyms
* expose more of the google search parameters

### in general
* provide interfaces for services and any other pluggable classes
