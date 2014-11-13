Minute PHP: PHP + AngularJS framework (v.001 alpha)

**Framework's USP:**

Built-in support for AngularJS

Router can inject models into controller (that can be accessed in AngularJS)

CRUD operations can be done with both PHP and AngularJS (with a single line on code)

**Project status**

It's highly experimental code! Just want to share the concept and code to see if anyone interested in collaborating.

**Demo**

Facebook like newsfeed in two minutes.

**Database setup:** Contains a user table, posts table and comments table, plus two more tables called PostsLikes and CommentsLikes.

**Understanding the router:**

$r->get($url, $controller, $acl, …$models)

The first three arguments are pretty standard:

1. $url: URL regex to match. Can contain placeholders. Placeholder format is ":name"
  1. Examples:
    1. /newsfeed/:user\_id/
    2. /newsfeed/:user\_id(/:page)?[here page parameter is optional]

2. $controller: format is similar to Laravel like "Filename@function"
  1. Example:
    1. Backend/Newsfeed@index [This will invoke the index function in Backend/Newsfeed class]

3. $acl: can be "true" for logged in otherwise false. Also supports strings like 'admin','power','business','trial' as defined in config file.
4. $models: This is the most unique part. 
  1. What are models?
    1. Models are basically PHP objects that support CRUD operations (in both PHP and AngularJS). 
    2. The model classes are automatically generated using a script. So for our sample database MinutePHP has generated 5 PHP Model classes called Users, Posts, Comments, PostsLikes and CommentsLikes.
    3. You can specify the Read, Update, Insert, and Delete permission for each model as "same\_user", "any\_user", "no\_user', "all", "none", "admin"

5. Example of model parameter
  1. $r->get('/newsfeed/:user\_id', '
