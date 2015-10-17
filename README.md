# angular-d3js

Angular service that implements D3.js - see http://d3js.org/

## Usage

Require the 'd3js' module in your app and inject the 'd3Service' into a directive.

Example:

``` javascript
// add the d3js module to your app
myapp = angular.module('myapp', ['d3js']);

// inject d3Service into your directive
(function() {
	'use strict';
	angular
		.module('myapp')
		.directive('myChart', myChart);

	function myChart(d3Service) {
		return {
			restrict: 'EA',
			scope: {
				data: '=',
			},
			template: '<div class="chart"></div>',
			link: function(scope, element, attrs) {
				// check angular-d3 module is loaded
				d3Service.load().then(function(d3) {
				
					// execute properly within Angular scope life cycle
					scope.safeApply = function(fn) {
						var phase = this.$root.$$phase;
						if(phase == '$apply' || phase == '$digest') {
							if(fn && (typeof(fn) === 'function')) { fn(); }
						} else {
						this.$apply(fn);
						}
					};

					// VIEWPORT
					/* component ie. directive == parentNode
					 * component controller == children[0]
					 * component div == children[0]
					 */
					var component = element[0].parentNode;
					var viewport = element[0].children[0].children[0];
					var svg = d3.select(viewport).append('svg');

					// render D3 chart on initialize and resize 
					scope.$watch(function(){
						var w = component.clientWidth;
						var h = component.clientHeight;
						return w + h;
					}, function() {
						scope.render();
					});

					// Example bar chart from http://bl.ocks.org/mbostock/7322386
					var data = [4, 8, 15, 16, 23, 42];
					// or use data from scope: var data = scope.data;

					var x = d3.scale.linear()
					.domain([0, d3.max(data)])
					.range([0, 420]);

					scope.render = function() {
						scope.safeApply( function() {
							svg.selectAll('*').remove();
							d3.svg
							.selectAll("div")
							.data(data)
							.enter().append("div")
							.style("width", function(d) { return x(d) + "px"; })
							.text(function(d) { return d; });
						});
					};
				});
			}
		};
	}
})();
```
