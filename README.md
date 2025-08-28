# ProjectMean 🚀

## MEAN Stack Deployment to Ubuntu in AWS

### Overview

This project implements a web solution based on the **MEAN stack** deployed on an Ubuntu server hosted on AWS.

The MEAN stack components are:

- **MongoDB**: Document-based NoSQL database storing application data as JSON-like documents.
- **Express.js**: Web application framework for backend API development.
- **AngularJS**: Frontend framework handling client-side logic and requests.
- **Node.js**: JavaScript runtime environment powering the server.

The app developed is a **Book Register web form** enabling CRUD operations for book records.

---

### Prerequisites

Before starting, ensure you have:

- AWS account with an Ubuntu EC2 virtual server (t2.micro recommended).
- Git Bash installed.
- SSH key pair for connecting to your EC2 instance via Git Bash.

---

### Step 1 — Launch your AWS EC2 instance

1. Log into your AWS console.
2. Select a region close to you.
3. Launch a new **t2.micro** Ubuntu server instance.
4. Create and securely store a new `.pem` private key.
5. Configure your security groups to allow:
   - SSH (port 22)
   - HTTP (port 80)
   - HTTPS (port 443) *(optional)*
<img width="1628" height="593" alt="image" src="https://github.com/user-attachments/assets/2f84781a-a86e-4df5-b35e-64b57d8a76e0" />

---

### Step 2 — Connect via SSH

cd downloads
ssh -i your-key.pem ubuntu@your-ec2-public-ip

<img width="829" height="219" alt="image" src="https://github.com/user-attachments/assets/991adae4-7da0-4025-a48a-c33a3c625ef2" />


---

### Step 3 — Install Node.js

sudo apt update
sudo apt upgrade
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

<img width="1208" height="835" alt="image" src="https://github.com/user-attachments/assets/e7125f1d-1f2c-4672-9951-8e68a073b751" />


---

### Step 4 — Install MongoDB

sudo apt-get install -y gnupg curl
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
sudo systemctl status mongod

<img width="1288" height="447" alt="image" src="https://github.com/user-attachments/assets/55d405ba-d1c3-4609-84cf-56b9e6bdab6c" />

---

### Step 5 — Setup npm and initialize project

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.bashrc
nvm install --lts
node -v
npm -v
sudo npm install body-parser
mkdir Books && cd Books
npm init -y

<img width="1191" height="871" alt="image" src="https://github.com/user-attachments/assets/34e3f2ed-fe0e-4580-ae96-d8af88f8fd33" />


---

### Step 6 — Create server.js

Create a `server.js` file with:

const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const path = require('path');
const app = express();
const PORT = process.env.PORT || 3300;

mongoose.connect('mongodb://localhost:27017/test', {
useNewUrlParser: true,
useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParser.json());

require('./apps/routes')(app);

app.listen(PORT, () => {
console.log(Server up: http://localhost:${PORT});
});

<img width="955" height="865" alt="image" src="https://github.com/user-attachments/assets/138d999e-8237-49e1-84ff-dd41b963876e" />


---

### Step 7 — Install Express and set up routes

sudo npm install express mongoose
mkdir -p apps models

text

Create `apps/routes.js`:

const Book = require('../models/book');
const path = require('path');

module.exports = function(app) {
app.get('/book', async (req, res) => {
try {
const books = await Book.find();
res.json(books);
} catch (err) {
res.status(500).json({ message: 'Error fetching books', error: err.message });
}
});

text
app.post('/book', async (req, res) => {
    try {
        const book = new Book(req.body);
        const savedBook = await book.save();
        res.status(201).json({ message: 'Successfully added book', book: savedBook });
    } catch (err) {
        res.status(400).json({ message: 'Error adding book', error: err.message });
    }
});

app.delete('/book/:isbn', async (req, res) => {
    try {
        const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
        if (!result) {
            return res.status(404).json({ message: 'Book not found' });
        }
        res.json({ message: 'Successfully deleted the book', book: result });
    } catch (err) {
        res.status(500).json({ message: 'Error deleting book', error: err.message });
    }
});

app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, '../public', 'index.html'));
});
};

<img width="1063" height="863" alt="image" src="https://github.com/user-attachments/assets/92d7717f-d3f9-4bd4-a34a-d9e671ebc604" />


Create `models/book.js`:

const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
name: { type: String, required: true },
isbn: { type: String, required: true, unique: true, index: true },
author: { type: String, required: true },
pages: { type: Number, required: true, min: 1 }
}, {
timestamps: true
});

module.exports = mongoose.model('Book', bookSchema);

<img width="862" height="853" alt="image" src="https://github.com/user-attachments/assets/7c59244b-9c13-4f3b-8f15-2fc97d29c5fd" />


---

### Step 8 — Setup AngularJS front-end

cd ../..
mkdir public && cd public

<img width="784" height="141" alt="image" src="https://github.com/user-attachments/assets/bb63181d-3eff-4b2c-aee3-e49015eb1c68" />


Create `scripts.js`:
vi scripts.js

angular.module('myApp', [])
.controller('myCtrl', function($scope, $http) {
function fetchBooks() {
$http.get('/book')
.then(response => {
$scope.books = response.data;
})
.catch(error => {
console.error('Error fetching books:', error);
});
}

fetchBooks();

$scope.del_book = function(book) {
    $http.delete(`/book/${book.isbn}`)
    .then(() => fetchBooks())
    .catch(console.error);
};

$scope.add_book = function() {
    const newBook = {
        name: $scope.Name,
        isbn: $scope.Isbn,
        author: $scope.Author,
        pages: $scope.Pages
    };

    $http.post('/book', newBook)
    .then(() => {
        fetchBooks();
        $scope.Name = $scope.Isbn = $scope.Author = $scope.Pages = '';
    })
    .catch(console.error);
};
});

<img width="1019" height="857" alt="image" src="https://github.com/user-attachments/assets/509f0157-a32b-4de4-8640-0ad70069956f" />


Create `index.html`:

<!DOCTYPE html> <html ng-app="myApp" ng-controller="myCtrl"> <head> <meta charset="UTF-8" /> <meta name="viewport" content="width=device-width, initial-scale=1" /> <title>Book Management</title> <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script> <script src="scripts.js"></script> <style> body { font-family: Arial, sans-serif; margin: 20px; } table { border-collapse: collapse; width: 100%; } th, td { border: 1px solid #ddd; padding: 8px; text-align: left; } th { background-color: #f2f2f2; } input[type="text"], input[type="number"] { width: 100%; padding: 5px; } button { margin-top: 10px; padding: 5px 10px; } </style> </head> <body> <h1>Book Management Application</h1>
text
<form ng-submit="add_book()">
    <label>Name:</label>
    <input type="text" ng-model="Name" required />

    <label>ISBN:</label>
    <input type="text" ng-model="Isbn" required />

    <label>Author:</label>
    <input type="text" ng-model="Author" required />

    <label>Pages:</label>
    <input type="number" ng-model="Pages" required />

    <button type="submit">Add Book</button>
</form>

<h2>Books</h2>

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>ISBN</th>
            <th>Author</th>
            <th>Pages</th>
            <th>Action</th>
        </tr>
    </thead>
    <tbody>
        <tr ng-repeat="book in books">
            <td>{{ book.name }}</td>
            <td>{{ book.isbn }}</td>
            <td>{{ book.author }}</td>
            <td>{{ book.pages }}</td>
            <td><button ng-click="del_book(book)">Delete</button></td>
        </tr>
    </tbody>
</table>
</body> </html> ```

<img width="1231" height="872" alt="image" src="https://github.com/user-attachments/assets/2ef6cdb4-f2ab-4e2c-822c-c259e2900817" />


Step 9 — Run the Application
text
cd ..
node server.js

<img width="1300" height="230" alt="image" src="https://github.com/user-attachments/assets/9e232b44-9b29-4c17-a2e2-8c45a263c306" />

To enable external access, open TCP port 3300 in your AWS security group.

Access the app in your browser at:

http://your_server_public_ip:3300
<img width="1917" height="646" alt="image" src="https://github.com/user-attachments/assets/1c9c9d8e-0b2e-4d1a-9dde-299fae07167b" />


Troubleshooting
If sudo service mongodbd start fails, use:

sudo systemctl start mongod

If sudo apt install npm throws errors, install npm via nvm:

text
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.bashrc
nvm install --lts

If compiling and starting your server fails with node server.js and throws this error 
![WhatsApp Image 2025-08-28 at 12 15 48_dfffe95b](https://github.com/user-attachments/assets/d4fe38b6-53ee-4e56-85d7-54deb82018a8)

Install express version 4 suing the command below.

npm install express@4.21.2

and it will work correctly
Also remember to always update the server before installing any service.
Congratulations!
You have successfully deployed and run a full MEAN stack application handling book records on AWS Ubuntu.
