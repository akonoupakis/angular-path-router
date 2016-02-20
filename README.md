# angular-path-router
> A url unaware deep link router

## Overview
angular-path-router creates routes that are driven by collections of path cases, suitable for creating routing applications that are not depended on url addresses

## Install

Install with [bower](http://bower.io/)

```sh
$ bower install angular-path-router
```

### Usage
All the deep link routers I have found are working conditionally by reading the deep link hash segment of the current url. This makes them unusuable when they are to be used dynamically without changing the url.

Consider this use case:

We have a list of managed products and for each product we want to be able to link related products.
Considering the user experience, we end up with something like this:
1) navigate to /products to view the list of products
2) click on a product and navigate to /products/{productId} to edit the product.
3) click on a link to open in a popup the /products endpoint in order for you to select the related products
4) optionally while in popup mode, you may edit a product before selecting it, or even create a new product to select (meaning that you need you need to navigate the popup from /products to /products/new and back to /products) 

With a url oriented router, you would have issues on steps 3 and 4, as the navigation function would change your main url hash.

angular-path-router draws the controllers and changes the url afterwards, rather than depend on it. You may specify if the router should navigate or not, and its possible for you to create multiple routers with the same specified path routes (for example, one router for the main navigation, and another router for each popup).

```js
(function () {
    
    angular.module("app", ['ngPathRouter'])
        .config(function($routerProvider) {
            
            //=> prepare our routes for the application
            $routerProvider
                .when('/', {
                    controller: 'HomeController',
                    template: '/tmpl/home.html'
                })
                .when('/products', {
                    controller: 'ProductsController',
                    template: '/tmpl/products.html'
                })
                .when('/products/:id', {
                    controller: 'ProductController',
                    template: '/tmpl/product.html'
                })
                .otherwise({
                    controller: 'NotFoundController',
                    template: '/tmpl/404.html'                    
                });
            
        })
        .controller('BaseController', function($scope, $router, $location) { //=> our base controller for our one page app 
             
             // create a routeId so that we know which of all routers is the one for this controller
             $scope.routeId = 'route-main';
             $scope.route = $router.create($scope.routeId, {
                path: $location.path(),
                redirect: true
             });
             
             $scope.$on('$destroy', function () {
                $router.dispose($scope.routeId);
             });
             
         });
        .controller('PopupController', function($scope, $router, $location) { //=> our base controller for our one page app 
             
             // create a routeId for this controller. Since there might be multiple popups, we need to create a unique id
             $router.$index = $router.$index || 0;
             $router.$index++;

             $scope.routeId = 'route-' + $router.$index;
                    
             $scope.route = $router.create($scope.routeId, {
                path: '/products', //=> the initial page path
                redirect: false //=> no need to adjust the location path here
             });
             
             $scope.$on('$destroy', function () {
                $router.dispose($scope.routeId);
             });
             
         });    
})();
```

Now, our main html should look like this
```html
<div ng-controller="BaseController">    
    <router-view route-id="routeId"></router-view>
</div>    
```

And our popup should open a code fraction such as:
```html
<div ng-controller="PopupController">    
    <router-view route-id="routeId"></router-view>
</div>    
```

## license

	The MIT License (MIT)

	Copyright (c) 2016 akon

	Permission is hereby granted, free of charge, to any person obtaining a copy
	of this software and associated documentation files (the "Software"), to deal
	in the Software without restriction, including without limitation the rights
	to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
	copies of the Software, and to permit persons to whom the Software is
	furnished to do so, subject to the following conditions:

	The above copyright notice and this permission notice shall be included in
	all copies or substantial portions of the Software.

	THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
	IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
	FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
	AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
	LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
	OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
	THE SOFTWARE.
