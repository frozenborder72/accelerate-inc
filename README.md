# Accelerate Inc - SEI Project 3

## Overview

Accelerate Inc is a Full-Stack MERN (MongoDB, Express, React and Node) that we created over a one week period in a group of three (with [Karim Ali](https://github.com/Karim999999999) and [Michael Stephanou](https://github.com/mstephanou)). It allows management of a team of athletes along with medical reports, training sessions, and it also has a dedicated blogging system.

![screenshot of the home page](public/img/Screenshot%202022-05-20%20at%2009.58.38.png)

This is the repo for the frontend, the backend can be found [here](https://github.com/frozenborder72/accelerate-inc-api)

## The Brief

- Build a full-stack application by making your own backend and your own front-end.
- Use an Express API to serve your data from a Mongo database.
- Consume your API with a separate front-end built with React.
- Be a complete product which most likely means multiple relationships and CRUD functionality for at least a couple of models.
- Implement thoughtful user stories/wireframes that are significant enough to help you know which features are core MVP and which you can cut.
- Be deployed online so it’s publicly accessible.

## Technologies Used

### Back-End

- Node.js
- MongoDB
- Mongoose
- Express
- Nodemon
- Bcrypt
- JSON Web Tokens

### Front-End

- React
- CSS
- Axios
- React-Router-Dom

### Development Tools

- Adobe XD (wireframing)
- Postman
- Git & GitHub
- Heroku (deployment)
- Netlify (deployment)
- VS Code

## Planning

We all agreed to build the app proposed by Karim, for which he also provided wireframes. We divided up the work by features. I was in charge of coding the articles and sessions (unfortunately unfinished due to lack of time) sections, and I also provided the main styling including the responsive navbar.
We also decided that we would have a daily standup on zoom to monitor the progress of the features we were assigned, plus extemporaneous meetings in case of need.

Some examples of the wireframes on which I based the CSS:
![sample of the wireframes](public/img/Screenshot%202022-05-10%20at%2012.13.46.png)
![sample of the wireframes](public/img/Screenshot%202022-05-10%20at%2012.11.54.png)

We also agreed that we would build both the backend and the frontend for each feature assigned.

Coding the articles managing section proved to be quite challenging, as the same components would present different views and allow different actions depending on the user logged in and his role. For example, only an editor could publish an article and have access to the entirety of the articles, while a user could only see and update his own. Also the filtering criteria by article status (draft, sent for review…) would change.

Example of the articles view for writer:
![Example of the articles view for writer](public/img/Screenshot%202022-05-20%20at%2010.10.11.png)

Example of the articles view for editor:
![Example of the articles view for editor](public/img/Screenshot%202022-05-20%20at%2010.13.20.png)

So I decided I would make a unique reusable add / update article form component, that would conditionally render the UI and allow different actions depending on whether the user was creating or updating an article, the user role (admin, editor…) and the article status (draft, published…)

Here’s a sample of the code that generates different buttons that call different functions depending on the aforementioned conditions.

```
const setActionButtons = (articleId, user) => {
    if (!articleId) {
      return (
        <div>
          <button type='submit' className='save btn' onClick={handleSubmit}>
            Save
          </button>
          <button
            type='button'
            className='set-to-editor btn'
            onClick={setArticleStatusToEditor}
          >
            Send to Editor
          </button>
        </div>
      );
    } else if (status === 'draft') {
      return (
        <div>
          <button
            type='submit'
            id='draft'
            className='save btn'
            onClick={modifyArticle}
          >
            Save
          </button>
          <button
            type='button'
            className='set-to-editor btn'
            onClick={setArticleStatusToEditor}
          >
            Send to Editor
          </button>
        </div>
      );
    } else if (status === 'editor' && user.isEditor) {
      return (
        <div>
          <button
            type='button'
            className='set-to-editor btn'
            onClick={setArticleStatusToPublished}
          >
            Publish
          </button>
          <button
            type='button'
            className='set-to-editor btn'
            onClick={setArticleStatusToReview}
          >
            Send For Review
          </button>
        </div>
      );
    } else if (status === 'editor') {
      return (
        <div>
          <button
            type='button'
            className='set-to-editor btn'
            onClick={setArticleStatusToDraft}
          >
            Cancel Publishing
          </button>
        </div>
      );
    } else if (status === 'published') {
      return (
        <div>
          <button
            type='button'
            className='set-to-editor btn'
            onClick={setArticleStatusToDraft}
          >
            Unpublish
          </button>
          <button
            type='button'
            className='set-to-editor btn'
            onClick={deleteArticle}
          >
            Delete Article
          </button>
        </div>
      );
    } else if (status === 'review' && user.isEditor) {
      return (
        <div>
          <Link
            className='btn'
            to={`/manage/show-message/${'Sending A Reminder To The Writer'}`}
          >
            Reminder
          </Link>
          <button
            type='button'
            className='set-to-editor btn'
            onClick={setArticleStatusToDraft}
          >
            Cancel Publishing
          </button>
        </div>
      );
    } else if (status === 'review') {
      return (
        <div>
          <button type='submit' className='save btn' onClick={handleSubmit}>
            Save
          </button>
          <button
            type='button'
            className='set-to-editor btn'
            onClick={setArticleStatusToEditor}
          >
            Send to Editor
          </button>
        </div>
      );
    }
  };
```

:
And for the general view I made a reusable table display component that would receive a table data prop that would change depending on the filtering criteria explained above.

```
{tableData.data.map(({ _id, title, createdAt, status }) => (
  <div className='table-card' key={_id}>
    <div className='table-item' id='title'>
      <h2>{title}</h2>
    </div>
    <div className='table-item' id='date'>
      <p>{createdAt}</p>
      <p>{status}</p>
    </div>
    <div className='table-item btn btn-view' id='viewbutton'>
      <Link
        className='button'
        to={`/manage/articles/${_id}/${status}`}
      >
        view
      </Link>
    </div>
  </div>
))}
```

In order to manage the complex filtering on the articles sessions, in the backend I implemented a custom middleware that would accept query parameters, and also provide pagination (not implemented, with regular expressions, used in order to modify the query strings so they would comply to express expected format.

```
const sortPaginate = model => async (req, res, next) => {
  let query;

  // Make a copy of req query in order to select only certain fields
  const reqQuery = { ...req.query };

  // Fields to esclude
  const fieldsToRemove = ['select', 'sort', 'page', 'limit'];

  // loop over fields to exclude and remove them from reqQuery
  fieldsToRemove.forEach(param => delete reqQuery[param]);

  // Create express friendly query strings, that will prepend a '$' to the operators
  const queryStr = JSON.stringify(reqQuery).replace(
    /\b(gt|gte|lt|lte|in)\b/g,
    match => `$${match}`
  );
```

As for the responsive menu, I struggled to implement the hamburger menu functionality. I ended up adding a custom js script (which is a big NO NO in React apps.) The solution was simply to set some boolean state that would be toggled on click event:

```
const [isMenuOpen, setIsMenuOpen] = useState(false);
  const location = useLocation();
  const navTheme = location.pathname.split('/')[1] || 'home';

  const toggleHamburger = () => setIsMenuOpen(isMenuOpen ? false : true);
  const closeMenuIfOpen = () => isMenuOpen && setIsMenuOpen(false);
```

## Wins & Blockers

### Wins

Building a full-stack (MERN) project for the first time.
Creating many relationships between back-end models to allow features such as complex filtering depending on the user role and what access they were given on a single feature (editors could modify the status of a draft article etc.)

### Blockers

In trying to implement the aforementioned complex relationships, we definitely pushed the non-relational nature of MongoDB a bit too far for our knowledge at the time, making it sometimes hard and time consuming to make the frontend work seamlessly with the backend.

### Bugs

Sometimes in Mongoose the filtering object passed to the Model.find function wouldn’t work as expected, so I’d have to implement filtering on the frontend.

### Future Improvements

Implementing a full responsive design and image upload for articles and athletes.

### Key Takeaways

Working in a group of three was a great way to learn about how to best communicate, plan and delegate responsibilities. Paradoxically enough, working on such a relationship heavy project on MongoDB well prepared me for future endeavours with Django/Postgres based projects.
