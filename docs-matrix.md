## Matrix API Docs

The Matrix API calculates the well known distance-matrix for a set of points, i.e. it calculates all the distances between every point combination. But we do not stop there, we also offer a time-, weight- and route-matrix. The weight-matrix can be used as raw input for e.g. a vehicle routing problem ([VRP](http://en.wikipedia.org/wiki/Vehicle_routing_problem)) and is more precise than a time- or distance-matrix. E.g. for bike routes the actual weight of a route (e.g. the "beauty") is what you want to decide if a route is 'better' and not always the taken time or distance. Also the weight is currently faster to calculate.

A simple illustration for a 3x3 matrix with identical points:

            |to_point1|to_point2|to_point3
:-----------|:--------|:--------|:--------
from_point1 |0        |1->2     | 1->3
from_point2 |2->1     |0        | 2->3
from_point3 |3->1     |3->2     | 0

A simple illustration for a 1x3 matrix with different start- and end-points:

            |to_point1   | to_point2 | t_point3
:-----------|:-----------|:----------|:--------
from_pointA |A->1        |A->2       |A->3


For every route 1->2, 1-3, ... or A->1,A->2,A->3 you can return only the weight, the time, the distance and even the full route. The matrix returning full routes is only suitable for a smaller matrix (or with a big matrix and a filter). This 'route-matrix' is useful if you know in advance that you need all the full routes and want to avoid a separate query to the Routing API. Useful e.g. for letting the user choosing from several routes. Routes itself can have several other parameters which are documented in our [Routing API documentation](https://github.com/graphhopper/web-api/blob/master/docs-routing.md).

## Parameters

All official parameters are shown in the following table

Parameter   | Default | Description
:-----------|:--------|:-----------
point       | -       | Specifiy multiple points for which the weight-, route-, time- or distance-matrix should be calculated. In this case the starts are identical to the destinations. If there are N points, then NxN entries will be calculated. The order of the point parameter is important. Specify at least three points. Cannot be used with `from_point` or `to_point.`
from_point  | -       | The starting points for the routes. E.g. if you want to calculate the three routes A->1, A->2, A->3 then you have one `from_point` parameter and three `to_point` parameters.
to_point    | -       | The destination points for the routes.
out_array   | weights   | Specifies which arrays should be included in the response. Specify one or more of the following options 'weights', 'times', 'distances', 'paths'. To specify more than one array use e.g. `out_array=times&out_array=distances`. The units of the entries of `distances` are meters, of `times` are seconds and of `weights` is arbitrary and it can differ for different vehicles or versions of this API.
vehicle     | car     | The vehicle for which the route should be calculated. Other vehicles are foot and bike
debug       | false   | If true, the output will be formated.

## Description



## Example output for the case type=json

Keep in mind that some attributes which are not documented here can be removed in the future - you should not rely on them! In the following example 4 points were specified and `out_array=distances&out_array=times&out_array=weights&debug=true`:

```json
{
  "weights" : [ [ 0.0, 449.753, 298.715, 664.113 ], [ 447.935, 0.0, 290.489, 543.242 ], [ 293.111, 283.954, 0.0, 452.725 ], [ 658.792, 537.769, 451.928, 0.0 ] ],
  "times" : [ [ 0, 5831, 3873, 8610 ], [ 5808, 0, 3766, 7043 ], [ 3800, 3681, 0, 5869 ], [ 8541, 6972, 5859, 0 ] ],
  "distances" : [ [ 0, 119124, 79156, 199907 ], [ 108691, 0, 63641, 142487 ], [ 79044, 71409, 0, 132215 ], [ 198852, 141322, 131953, 0 ] ],
  "info" : {
    "copyrights" : [ "GraphHopper", "OpenStreetMap contributors" ],
    "took" : 0.024203006
  }
}
```

The JSON result contains the following structure:

JSON path/attribute        | Description
:--------------------------|:------------
times                      | The time matrix for the specified points in the order [[from1->to1, from1->to2, ...], [from2->to1, from2->to2, ...], ...]. The times are in seconds.
distances                  | The distance matrix for the specified points in the same order as the time matrix. The distances are in meters.
weights                    | The weight matrix for the specified points in the same order as the time matrix. The weights for different vehicles can have a different number but are perfectly suited as input for [Vehicle Routing Problems](http://en.wikipedia.org/wiki/Vehicle_routing_problem) as they are usually faster to calculate.
info.took                  | The taken time in seconds
info.copyrights            | For the FREE and STANDARD package attribution according to [our documentation](https://github.com/graphhopper/web-api/#attribution) is necessary.


### Output if expected error(s) while creating the matrix:
```json
{
  "info" : {
    "errors" : [ {
      "message" : "Cannot find from_points: 7, 8"
    }, {
      "message" : "Cannot find to_points: 7, 8"
    } ],
    "copyrights" : [ "GraphHopper", "OpenStreetMap contributors" ],
    "took" : 0.002572816
  }
}
```


JSON path/attribute    | Description
:----------------------|:------------
info.errors            | If one point cannot be found then GraphHopper will skip calculating and print a message ala "Cannot find from_points". Make sure to correct or remove it.


### HTTP Error codes

HTTP error code | Reason
:---------------|:------------
500             | Internal server error. We get automatically a notification and will try to fix this fast.
501 	    | Only a special list of vehicles is supported
400 	    | Something was wrong in your request