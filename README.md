#MinutePHP: PHP + AngularJS framework (*v.001*)#

##Framework's USP:##
- Built-in support for AngularJS
- Router can inject models into controller (that can be accessed in AngularJS)
- CRUD operations can be done with both PHP and AngularJS (with a single line on code)

##Project status##
It's highly experimental code! Just want to share the concept and code to see **if anyone interested in collaborating!


----------


**

###Facebook like newsfeed demo (in two minutes)###

**Database setup:** Contains a user table, posts table and comments table, plus two more tables called PostsLikes and CommentsLikes.

**Understanding the new Router syntax:**

```php 
$r->get($url, $controller, $acl, …$models)
```

The first three arguments are pretty standard:

1. *$url:* URL regex to match. Can contain placeholders (or regex). Placeholder format is ":name"
  1. Examples:
     1. /newsfeed/:user\_id/
     2. /newsfeed/:user\_id(/:page)? *[here page parameter is optional]*

2. *$controller:* format is similar to Laravel like "Filename@function"
  1. Example:
     1. Backend/Newsfeed@index *[This will invoke the index function in Backend/Newsfeed class]*

3. *$acl:* can be "true" for logged in otherwise false. Also supports strings like 'admin','power','business','trial' as defined in config file.
4. **$model objects: This is the most unique part. **
  1. What are model objects?
     1. Models are basically PHP objects that support CRUD operations (in both PHP and AngularJS). Something similar to an ORM but not that (as explained later).
     2. The model classes are automatically generated using a script. So for our sample database MinutePHP has generated 5 PHP Model classes called *Users, Posts, Comments, PostsLikes and CommentsLikes*
     3. You can specify the Read, Update, Insert, and Delete permission for each model as "same\_user", "any\_user", "all", "none", "admin", etc (explained later)

5. **The Router can inject Model objects in to the controller** automatically. 

**Here is some sample code:**

**index.php:**
```php
//$r is instance of Router
//Sample route to match /newsfeed/1, /newsfeed/2, etc (digit => :user_id)
$r->get('/newsfeed/:user_id', 'Backend/Newsfeed@index', true, 'posts[user_id][]', 'users[posts.user_id] as poster')
```

Explanation of the *last two parameters*:
   
1. ***posts[user_id][]*** - this will load all posts with user_id matching *:user_id* in URL (our placeholder)
   1.  It will then create `$posts` object (instance of Posts model) which is an array of `post` items.
   2.  It will pass the `$posts` object to the controller (see below). 
   3. Example: /newsfeed/1 will fill `$posts` with post items where `user_id=1`.
   4. The **[ ]** at end is to specify we want an array or *all* matching results.
2.  ***users[posts.user_id] as poster*** - this will query the Users model using `user_id` from $posts. As a result it will:
   1. Create a $poster object (instance of User model) and pass it to the controller (see below). 
   2. In addition to that it will also add a references inside \$posts for each post item.
   3. If it is getting confusing, think of it like an SQL query:
```SELECT * FROM posts as A, users as b where user_id = :user_id and b.user_id = a.user_id```
   4. But don't worry too much about this. It will become clear with this example.
  5. Note that there is no [] at the end, which means this time it will only load the first matching record (instead of an array of objects)

**Backend/Newsfeed class:**
```php
//This is called by the Router when a person accesses /newsfeed/1, /newsfeed/2, etc in browser

public function index($posts, $poster) {

    //Render Backend/Newsfeed view located in Views folder 
    //Magically create an AngularJS object for $posts

	View::make('Backend/Newsfeed', $posts); 
}
```
**Views/Backend/Newsfeed: (view)**
```html
Hello!

Write something:
<input type="text" ng-model="newtitle" />
<button ng-click="createPost();">Add post</button>
	
//posts is now an AngularJS object
<div ng-repeat="post in posts">
	<h3>{{post.title}}</h3> by post.poster.first
	
	<input type="text" ng-model="mytitle" value="changed title" />
	<button ng-click="updatePost(post);">change</button>
</div>

<script>
function extend($scope) {
	$scope.createPost = function() {
	    //will create a new post and with the user's title
		var post = $scope.posts.create().set('title', $scope.newtitle).save();
		console.log("post: ", post);
	}
		
	$scope.updatePost = function(item) {
		//will change the post's title to user input
		item.set('title', $scope.mytitle).save();
	}
}
</script>
```

###So are you with me so far? :)###

As you can see above, the View class automatically converted our $posts Model into an AngularJS object and added it to our current `$scope` as `$scope.posts`

`$scope.posts` is an array of objects on which you can iterate with ng-repeat.

`$scope.posts` is an array but it has a few functions of its own like `.create()`, `loadNextPage()`, etc. 

Each item in `$scope.posts` array (say, `PostItem`) is an object with functions like `set()`, `save()`, etc.

**But here is the most interesting part!**

Do you remember about our `users[posts.user_id] as poster` in the router? 

What this means to you is that we created a `$posts` array. Then we created `$poster` object (instance of Users Model) and loaded results from the users table by matching the `user_id`.

But what this also means to you is that each item inside `$scope.posts` now also contains a reference to it's `poster` object as well.

So `{{$posts[0].poster.first}}` will print the first name of the poster in AngularJS.

>Too hard to comprehend? I hope not! :)

Ok, let's take a look at this route.. it's may look tough but the concept behind it is simple and once you get it, things will become very clear:

```php
$r->get('/newsfeed/:user_id', 'Backend/Newsfeed@index', false,

    'posts[user_id][2] order by post_id asc', 'users[posts.user_id] as poster',

    'postsLikes[posts.post_id][] as plikes', 'users[plikes.user_id] as pliker',

    'comments[posts.post_id][] order by comment_id desc', 'users[comments.user_id] as commenter',

    'commentsLikes[comments.comment_id][] as clikes', 'users[clikes.user_id] as cliker'
);
```

Now let's pull our parameters apart:

1. `posts[user_id][2] order by post_id asc` Same as before only difference from last time is
   1. We load only the first two records, hence the [2] instead of [] at the end. 
   2. We can load more records inside AngularJS on the `$scope.posts` by calling `$scope.posts.loadNextPage()` as you will see in the example below.
   3. You can also use posts[user_id][1,2] which will load the second page (skip the first two records)
2.  `users[posts.user_id] as poster` Same as last time. We want to load the poster of each post by creating a join on posts.user_id on the users table. Remember that since we don't have a trailing [] at the end, this is an object not an array.
3. `postsLikes[posts.post_id][] as plikes` Create a `$postsLikes` object by creating a join on `posts.post_id` on the post_likes table. This time we do have a trailing [] at the end, so what this means is that we will get an array instead of single object.
4. `users[plikes.user_id] as pliker` We want to know who liked the posts. So we create a $pliker object creating a join on `plikes.user_id` (i.e. `postsLikes.user_id`) on the users table. This is not an array but a single object.
5. `comments[posts.post_id][] order by comment_id desc` Since each posts can have comments inside it, we create a $comments object joining `post_id` on `posts` and `comments`. This will also return an array. 
   6. Since we wish to load the comments in the reverse order in which they were posted, we add an order by clause similar to SQL.
6. `users[comments.user_id] as commenter` We surely want to know who made the comment, so we create a $commenter object joining the `user_id` on the `comments` and `users` table.
7. `commentsLikes[comments.comment_id][] as clikes` This is pretty much same as postLikes, except we're using it for comments.



