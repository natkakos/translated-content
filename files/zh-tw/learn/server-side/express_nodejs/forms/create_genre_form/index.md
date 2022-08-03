---
title: 創建種類表單
slug: Learn/Server-side/Express_Nodejs/forms/Create_genre_form
translation_of: Learn/Server-side/Express_Nodejs/forms/Create_genre_form
---
<p>本章節演示如何定義我們的頁面，創建<code>Genre</code> 物件（這是一個很好的起點，因為類型<code>Genre</code>只有一個欄位，就是它的名稱 <code>name</code>，沒有依賴項）。像任何其他頁面一樣，我們需要設置路由，控制器和視圖。</p>

<p> </p>

<h2 id="引入驗證與無害化方法">引入驗證與無害化方法</h2>

<p>在我們的控制器中使用 <em>express-validator</em> 驗證器，我們必須導入我們想要從 '<strong>express-validator/check</strong>' 和 '<strong>express-validator/filter</strong>' 模塊中使用的函數。</p>

<p>打開 <strong>/controllers/genreController.js</strong>，並在文件頂部添加以下幾行：</p>

<pre class="brush: js line-numbers  language-js"><code class="language-js">const { body,validationResult } = require('express-validator/check');
const { sanitizeBody } = require('express-validator/filter');</code></pre>

<h2 id="Controller—get_route">Controller—get route</h2>

<p>Find the exported <code>genre_create_get()</code> controller method and replace it with the following code. This simply renders the <strong>genre_form.pug</strong> view, passing a title variable.</p>

<pre class="brush: js line-numbers  language-js"><code class="language-js">// Display Genre create form on GET.
exports.genre_create_get = function(req, res, next) {
    res.render('genre_form', { title: 'Create Genre' });
};</code></pre>

<h2 id="Controller—post_route">Controller—post route</h2>

<p>Find the exported <code>genre_create_post()</code> controller method and replace it with the following code.</p>

<pre class="brush: js line-numbers  language-js"><code class="language-js">// Handle Genre create on POST.
exports.genre_create_post =  [

    // Validate that the name field is not empty.
    body('name', 'Genre name required').isLength({ min: 1 }).trim(),

    // Sanitize (trim and escape) the name field.
    sanitizeBody('name').trim().escape(),

    // Process request after validation and sanitization.
    (req, res, next) =&gt; {

        // Extract the validation errors from a request.
        const errors = validationResult(req);

        // Create a genre object with escaped and trimmed data.
        var genre = new Genre(
          { name: req.body.name }
        );


        if (!errors.isEmpty()) {
            // There are errors. Render the form again with sanitized values/error messages.
            res.render('genre_form', { title: 'Create Genre', genre: genre, errors: errors.array()});
        return;
        }
        else {
            // Data from form is valid.
            // Check if Genre with same name already exists.
            Genre.findOne({ 'name': req.body.name })
                .exec( function(err, found_genre) {
                     if (err) { return next(err); }

                     if (found_genre) {
                         // Genre exists, redirect to its detail page.
                         res.redirect(found_genre.url);
                     }
                     else {

                         genre.save(function (err) {
                           if (err) { return next(err); }
                           // Genre saved. Redirect to genre detail page.
                           res.redirect(genre.url);
                         });

                     }

                 });
        }
    }
];</code></pre>

<p>The first thing to note is that instead of being a single middleware function (with arguments <code>(req, res, next)</code>) the controller specifies an <em>array</em> of middleware functions. The array is passed to the router function and each method is called in order.</p>

<ul>
</ul>

<div class="note">
<p><strong>Note:</strong> This approach is needed, because the sanitisers/validators are middleware functions.</p>
</div>

<p>The first method in the array defines a validator (<code>body</code>) to check that the <em>name</em> field is not empty (calling <code>trim()</code> to remove any trailing/leading whitespace before performing the validation). The  second method in the array (<code>sanitizeBody()</code>) creates a sanitizer to <code>trim()</code> the <em>name</em> field and <code>escape()</code> any dangerous  HTML characters.</p>

<pre class="brush: js line-numbers  language-js"><code class="language-js">// Validate that the name field is not empty.
body('name', 'Genre name required').isLength({ min: 1 }).trim(),

// Sanitize (trim and escape) the name field.
sanitizeBody('name').trim().escape(),</code></pre>

<ul>
</ul>

<div class="note">
<p><strong>Note:</strong> Sanitizers run during validation do not modify the request. That is why we have to call <code>trim()</code> in both steps above!</p>
</div>

<p>After specifying the validators and sanitizers we create a middleware function to extract any validation errors. We use <code>isEmpty()</code> to check whether there are any errors in the validation result. If there are then we render the form again, passing in our sanitised genre object and the array of error messages (<code>errors.array()</code>).</p>

<pre class="brush: js line-numbers  language-js"><code class="language-js">// Process request after validation and sanitization.
(req, res, next) =&gt; {

    // Extract the validation errors from a request.
    const errors = validationResult(req);

    // Create a genre object with escaped and trimmed data.
    var genre = new Genre(
      { name: req.body.name }
    );

    if (!errors.isEmpty()) {
        // There are errors. Render the form again with sanitized values/error messages.
        res.render('genre_form', { title: 'Create Genre', genre: genre, errors: errors.array()});
    return;
    }
    else {
        // Data from form is valid.
        ... &lt;save the result&gt; ...
    }
}</code></pre>

<p>If the genre name data is valid then we check if a <code>Genre</code> with the same name already exists (as we don't want to create duplicates). If it does we redirect to the existing genre's detail page. If not, we save the new <code>Genre</code> and redirect to its detail page.</p>

<pre class="brush: js line-numbers  language-js"><code class="language-js">// Check if Genre with same name already exists.
Genre.findOne({ 'name': req.body.name })
    .exec( function(err, found_genre) {
    if (err) { return next(err); }
        if (found_genre) {
            // Genre exists, redirect to its detail page.
            res.redirect(found_genre.url);
            }
        else {
            genre.save(function (err) {
                if (err) { return next(err); }
                    // Genre saved. Redirect to genre detail page.
                    res.redirect(genre.url);
                });
        }
});</code></pre>

<p>This same pattern is used in all our post controllers: we run validators, then sanitisers,  then check for errors and either re-render the form with error information or save the data. </p>

<h2 id="View">View</h2>

<p>The same view is rendered in both the <code>GET</code> and <code>POST</code> controllers/routes when we create a new <code>Genre</code> (and later on it is also used when we <em>update</em> a <code>Genre</code>). In the <code>GET</code> case the form is empty and we just pass a title variable. In the <code>POST</code> case the user has previously entered invalid data—in the <code>genre</code> variable we pass back a sanitized version of the entered data and in the <code>errors</code> variable we pass back an array of error messages.</p>

<pre class="brush: js line-numbers  language-js"><code class="language-js">res.render('genre_form', { title: 'Create Genre'});
res.render('genre_form', { title: 'Create Genre', genre: genre, errors: errors.array()});</code></pre>

<p>Create <strong>/views/genre_form.pug</strong> and copy in the text below.</p>

<pre class="brush: html"><code class="language-html">extends layout

block content
  h1 #{title}

  form(method='POST' action='')
    div.form-group
      label(for='name') Genre:
      input#name.form-control(type='text', placeholder='Fantasy, Poetry etc.' name='name' value=(undefined===genre ? '' : genre.name))
    button.btn.btn-primary(type='submit') Submit

  if errors
    ul
      for error in errors
        li!= error.msg</code></pre>

<p>Much of this template will be familiar from our previous tutorials. First we extend the <strong>layout.pug</strong> base template and override the <code>block</code> named '<strong>content</strong>'. We then have a heading with the <code>title</code> we passed in from the controller (via the <code>render()</code> method).</p>

<p>Next we have the pug code for our HTML form that uses the <code>POST</code> <code>method</code> to send the data to the server, and because the <code>action</code> is an empty string, will send the data to the same URL as the page.</p>

<p>The form defines a single required field of type "text" called "name". The default <em>value</em> of the field depends on whether the <code>genre</code> variable is defined. If called from the <code>GET</code> route it will be empty as this is a new form. If called from a <code>POST</code> route it will contain the (invalid) value originally entered by the user.</p>

<p>The last part of the page is the error code. This simply prints a list of errors, if the error variable has been defined (in other words, this section will not appear when the template is rendered on the <code>GET</code> route).</p>

<div class="note">
<p><strong>Note:</strong> This is just one way to render the errors. You can also get the names of the affected fields from the error variable, and use these to control where the error messages are rendered, whether to apply custom CSS, etc.</p>
</div>

<h2 id="What_does_it_look_like">What does it look like?</h2>

<p>Run the application, open your browser to <a href="http://localhost:3000/">http://localhost:3000/</a>, then select the <em>Create new genre </em>link. If everything is set up correctly, your site should look something like the following screenshot. After you enter a value, it should be saved and you'll be taken to the genre detail page.</p>

<p><img alt="Genre Create Page - Express Local Library site" src="locallibary_express_genre_create_empty.png"></p>

<p>The only error we validate against server-side is that the genre field must not be empty. The screenshot below shows what the error list would look like if you didn't supply a genre (highlighted in red).</p>

<p><img src="locallibary_express_genre_create_error.png"></p>

<div class="note">
<p><strong>備註：</strong> Our validation uses <code>trim()</code> to ensure that whitespace is not accepted as a genre name. We can also validate that the field is not empty on the client side by adding the value <code>required='true'</code> to the field definition in the form:</p>

<pre class="brush: js"><code>input#name.form-control(type='text', placeholder='Fantasy, Poetry etc.' name='name' value=(undefined===genre ? '' : genre.name), <strong>required='true'</strong> )</code></pre>
</div>

<h2 id="Next_steps">Next steps</h2>

<ul>
 <li>Return to <a href="/en-US/docs/Learn/Server-side/Express_Nodejs/forms">Express Tutorial Part 6: Working with forms.</a></li>
 <li>Proceed to the next subarticle of part 6: <a href="/en-US/docs/Learn/Server-side/Express_Nodejs/forms/Create_author_form">Create Author form</a>.</li>
</ul>