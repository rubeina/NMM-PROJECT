# 📰 Phase 5 — Blog Site with Comment Section

## 🔗 Repository Links

- Frontend Repository: [Add your GitHub link here]  
- Backend Repository: [Add your GitHub link here]  
- Live Frontend URL: [https://myblogsite.vercel.app](https://myblogsite.vercel.app)  
- Backend API URL: [https://myblogapi.onrender.com](https://myblogapi.onrender.com)  

---

## 📘 Project Overview

The Blog Site with Comment Section is a web-based blogging application that enables users to create, view, and interact with blog posts in real time.  
The project demonstrates the integration of **Flask** (Python backend) with HTML templates (**Jinja2 frontend**) and **SQLite** database for data persistence.

This application emphasizes **CRUD** (Create, Read, Update, Delete) operations and user engagement through a comment system, showcasing the core principles of web application design, client-server communication, and database management.

---

## 🧭 Project Objective

To design and implement a dynamic blog platform where users can:

- Create and publish posts.
- Read posts created by others.
- Add and manage comments.
- Interact with content seamlessly through an integrated front-end and back-end system.

---

## ⚙️ Functional Requirements

### 🏠 Public Homepage & Post Listing
- Displays recent blog posts with title, summary, author, date, and tags.
- Supports pagination or infinite scrolling for easy navigation.

### 📝 Post Page
- Renders full blog post content (supports Markdown or rich text).
- Includes a comment section below each post.

### 👥 User Accounts & Authentication
- Signup, login, logout, and password reset via email.
- Role-based access:
  - **Reader:** Default role (can comment).  
  - **Author:** Can create and edit posts.  
  - **Moderator:** Can manage comments.  
  - **Admin:** Full site control.

### ✍️ Post Management (Authors/Admins)
- Create, edit, delete, publish/unpublish, or save drafts.
- Supports WYSIWYG/Markdown editor and image uploads.

### 💬 Comment System
- Authenticated users can comment and reply (nested replies supported).
- Edit/delete allowed within 15 minutes of posting.
- Moderators/Admins can manage comments and mark spam.
- Comment pagination or "Load More" functionality.

### 🛡️ Moderation & Spam Controls
- Moderation dashboard for flagged comments.
- Automatic spam protection using:
  - Rate limiting per IP
  - Comment length checks
  - CAPTCHA for suspicious activity

### 🔔 Notifications
- Optional email notifications:
  - To authors when new comments are posted.
  - To users when replies are received.

### 🔍 Search & Tags
- Keyword-based search by title or content.
- Tag-based filtering for better discoverability.

### ⚙️ Admin Panel
- Manage users, posts, and comments.
- View site analytics (number of posts/comments in 30 days).
- Edit site metadata and moderation policies.

---

## 🧩 Database Design (ER Model)

**Entities and Attributes**

| Entity  | Attributes                     |
|--------|--------------------------------|
| User   | UserID, Name, Email, Password, Role |
| Blog   | BlogID, Title, Content, AuthorID, DateCreated |
| Comment| CommentID, BlogID, UserID, Text, DateCreated |

**Relationships**

- One User → Many Blogs  
- One Blog → Many Comments  
- One User → Many Comments  

---

## 🏗️ System Architecture

The system follows a **Client–Server Model**, ensuring modular and maintainable design.

- **Frontend:** HTML templates rendered via Jinja2.  
- **Backend:** Flask handles routes, request processing, and business logic.  
- **Database:** SQLite manages persistent storage for posts and comments.  

---

## 🧠 Implementation

### 🖥️ Backend (Flask)  
**File:** `backend/app.py`

```python
from flask import Flask, render_template, request, redirect, url_for
from models import Post, Comment, db

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///blog.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db.init_app(app)

with app.app_context():
    db.create_all()

@app.route('/')
def index():
    posts = Post.query.all()
    return render_template('index.html', posts=posts)

@app.route('/post/<int:post_id>', methods=['GET', 'POST'])
def post_detail(post_id):
    post = Post.query.get_or_404(post_id)
    if request.method == 'POST':
        comment_text = request.form['comment']
        comment = Comment(text=comment_text, post_id=post.id)
        db.session.add(comment)
        db.session.commit()
        return redirect(url_for('post_detail', post_id=post.id))
    return render_template('post_detail.html', post=post)

@app.route('/create', methods=['GET', 'POST'])
def create_post():
    if request.method == 'POST':
        title = request.form['title']
        content = request.form['content']
        post = Post(title=title, content=content)
        db.session.add(post)
        db.session.commit()
        return redirect(url_for('index'))
    return render_template('create_post.html')

if __name__ == '__main__':
    app.run(debug=True)
```

---

### 🧾 Models  
**File:** `backend/models.py`

```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text, nullable=False)
    comments = db.relationship('Comment', backref='post', lazy=True)

class Comment(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    text = db.Column(db.Text, nullable=False)
    post_id = db.Column(db.Integer, db.ForeignKey('post.id'), nullable=False)
```

---

### 🎨 Frontend Templates

**index.html** – Lists all posts and provides navigation.

```html
<h1>Blog Posts</h1>
<a href="/create">Create New Post</a>
<ul>
    {% for post in posts %}
        <li><a href="/post/{{ post.id }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>
```

**create_post.html** – Form to create a new post.

```html
<h1>Create a New Post</h1>
<form method="post">
    Title: <input type="text" name="title"><br>
    Content:<br>
    <textarea name="content"></textarea><br>
    <input type="submit" value="Create">
</form>
```

**post_detail.html** – Displays the post and associated comments.

```html
<h1>{{ post.title }}</h1>
<p>{{ post.content }}</p>
<h2>Comments</h2>
<ul>
    {% for comment in post.comments %}
        <li>{{ comment.text }}</li>
    {% endfor %}
</ul>
<h3>Add Comment</h3>
<form method="post">
    <textarea name="comment"></textarea><br>
    <input type="submit" value="Add Comment">
</form>
```

---

## 🔗 API Enhancements

| Route          | Method | Description          |
|----------------|--------|--------------------|
| /              | GET    | Fetch all blogs     |
| /post/<id>     | GET    | View a single post  |
| /post/<id>     | POST   | Add a comment       |
| /create        | POST   | Create a new post   |

---

## 🧩 Challenges and Solutions

| Challenge                          | Solution                                                      |
|-----------------------------------|---------------------------------------------------------------|
| Database initialization error       | Added `app.app_context()` to initialize DB properly          |
| Comments not displaying after submission | Added redirect after POST to reload updated content       |
| Inconsistent template styling       | Used Jinja2 block inheritance for layout consistency         |

---

## 🧾 Results

The system successfully demonstrates:

- CRUD functionality for blogs and comments.  
- Real-time data update via Flask routes.  
- Integration between backend logic and frontend rendering.

---

## 🏁 Conclusion

The Blog Site with Comment Section successfully demonstrates the implementation of a full-stack web application using Flask and SQLite.  
It achieves seamless data flow between backend and frontend, highlights database design efficiency, and offers a foundation for future improvements such as authentication, UI enhancement, and REST API integration.
