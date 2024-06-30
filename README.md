
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Walletcracker</title>

    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/daisyui@2.51.6/dist/full.css" rel="stylesheet" type="text/css" />
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.6.8/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.6.8/firebase-firestore.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.6.8/firebase-auth.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.6.8/firebase-storage.js"></script>
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
</head>
<body class="bg-base-200">
    <div id="app" class="container mx-auto my-8">
        <div v-if="!user" class="flex justify-center">
            <div class="card bg-base-100 shadow-xl w-full max-w-md">
                <div class="card-body">
                    <h2 class="card-title text-2xl font-bold mb-4 text-center">Walletcracker</h2>
                    <div v-if="showLoginForm">
                        <div class="form-control mb-4">
                            <label for="loginEmail" class="label">Email</label>
                            <input type="email" id="loginEmail" v-model="loginEmail" class="input input-bordered">
                        </div>
                        <div class="form-control mb-6">
                            <label for="loginPassword" class="label">Password</label>
                            <input type="password" id="loginPassword" v-model="loginPassword" class="input input-bordered">
                        </div>
                        <button @click="login" class="btn btn-primary btn-block">
                            <span class="material-icons">login</span> Login
                        </button>
                        <button @click="loginAnonymously" class="btn btn-ghost btn-block mt-2">
                            <span class="material-icons">person_outline</span> Login as Guest
                        </button>
                    </div>
                    <div v-else>
                        <div class="form-control mb-4">
                            <label for="signupEmail" class="label">Email</label>
                            <input type="email" id="signupEmail" v-model="signupEmail" class="input input-bordered">
                        </div>
                        <div class="form-control mb-6">
                            <label for="signupPassword" class="label">Password</label>
                            <input type="password" id="signupPassword" v-model="signupPassword" class="input input-bordered">
                        </div>
                        <button @click="signup" class="btn btn-success btn-block">
                            <span class="material-icons">person_add</span> Signup
                        </button>
                    </div>
                    <div class="text-center mt-6">
                        <button @click="toggleForm" class="link link-primary">
                            <span class="material-icons">{{ showLoginForm ? 'person_add' : 'login' }}</span> {{ showLoginForm ? 'Switch to Signup' : 'Switch to Login' }}
                        </button>
                    </div>
                </div>
            </div>
        </div>
        <div v-else>
            <div class="navbar bg-base-100 shadow-lg mb-6">
                <div class="flex-1">
                    <h1 class="text-2xl font-bold">Walletcracker</h1>
                </div>
                <div class="flex-none">
                    <button @click="logout" class="btn btn-primary">
                        <span class="material-icons">logout</span> Logout
                    </button>
                </div>
            </div>
            <div class="card bg-base-100 shadow-xl mb-6">
                <div class="card-body">
                    <div class="form-control mb-4">
                        <label for="imageInput" class="label">Upload Image</label>
                        <input type="file" id="imageInput" @change="handleImageUpload" class="file-input file-input-bordered w-full">
                    </div>
                    <button @click="uploadImage" class="btn btn-primary">
                        <span class="material-icons">cloud_upload</span> Upload
                    </button>
                </div>
            </div>
            <div class="grid grid-cols-3 gap-4">
                <div v-for="image in images" :key="image.id" class="relative">
                    <img :src="image.url" class="w-full h-64 object-cover rounded-lg">
                    <div class="absolute top-2 right-2">
                        <button @click="deleteImage(image.id)" class="btn btn-error btn-sm btn-circle">
                            <span class="material-icons">delete</span>
                        </button>
                    </div>
                    <div>
                        <textarea class="textarea textarea-bordered w-full mt-2" v-model="commentText" placeholder="Add a comment"></textarea>
                        <button @click="addComment(image.id)" class="btn btn-primary btn-sm mt-2">
                            <span class="material-icons">send</span> Submit
                        </button>
                        <div v-for="comment in image.comments" :key="comment.id" class="mt-2">
                            <p class="font-semibold">{{ comment.name }}</p>
                            <p>{{ comment.text }}</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <script>
        var firebaseConfig = {};

        firebase.initializeApp(firebaseConfig);

        const auth = firebase.auth();
        const db = firebase.firestore();
        const storage = firebase.storage();

        new Vue({
            el: '#app',
            data: {
                user: null,
                showLoginForm: true,
                loginEmail: '',
                loginPassword: '',
                signupEmail: '',
                signupPassword: '',
                currentImage: null,
                images: [],
                commentText: ''
            },
            methods: {
                login() {
                    auth.signInWithEmailAndPassword(this.loginEmail, this.loginPassword)
                        .then((userCredential) => {
                            this.user = userCredential.user;
                            this.fetchImages();
                        });
                },
                loginAnonymously() {
                    auth.signInAnonymously()
                        .then(() => {
                            this.user = auth.currentUser;
                            this.fetchImages();
                        });
                },
                signup() {
                    auth.createUserWithEmailAndPassword(this.signupEmail, this.signupPassword)
                        .then((userCredential) => {
                            this.user = userCredential.user;
                            this.fetchImages();
                        });
                },
                toggleForm() {
                    this.showLoginForm = !this.showLoginForm;
                },
                logout() {
                    auth.signOut()
                        .then(() => {
                            this.user = null;
                            this.images = [];
                        });
                },
                handleImageUpload(event) {
                    this.currentImage = event.target.files[0];
                },
                uploadImage() {
                    if (this.currentImage) {
                        const storageRef = storage.ref(`images/${this.user.uid}/${this.currentImage.name}`);
                        storageRef.put(this.currentImage)
                            .then(() => {
                                storageRef.getDownloadURL()
                                    .then((url) => {
                                        db.collection('images').add({
                                            url: url,
                                            userId: this.user.uid,
                                            comments: []
                                        })
                                        .then(() => {
                                            this.currentImage = null;
                                            this.fetchImages();
                                        });
                                    });
                            });
                    }
                },
                fetchImages() {
                    db.collection('images').get()
                        .then((querySnapshot) => {
                            this.images = querySnapshot.docs.map((doc) => ({
                                id: doc.id,
                                ...doc.data()
                            }));
                        });
                },
                deleteImage(id) {
                    db.collection('images').doc(id).delete()
                        .then(() => {
                            this.fetchImages();
                        });
                },
                addComment(imageId) {
                    const imageRef = db.collection('images').doc(imageId);
                    imageRef.get()
                        .then((doc) => {
                            const comments = doc.data().comments || [];
                            comments.push({
                                name: this.user ? this.user.email : 'Guest',
                                text: this.commentText
                            });
                            imageRef.update({
                                comments: comments
                            })
                            .then(() => {
                                this.commentText = '';
                                this.fetchImages();
                            });
                        });
                }
            },
            created() {
                auth.onAuthStateChanged((user) => {
                    this.user = user;
                    if (user) {
                        this.fetchImages();
                    }
                });
            }
        });
    </script>
</body>
</html>
```
