<template>
    <div class="max-w-4xl mx-auto p-4">
        <h1 class="text-3xl font-bold mb-6">Posts</h1>
        <form class="mb-6 bg-white p-4 shadow-md rounded-md" @submit.prevent="savePost" enctype="multipart/form-data">
            <div class="mb-4">
                <input type="text" v-model="form.title" placeholder="Title" class="w-full p-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-indigo-300"/>
            </div>
            <div class="mb-4">
                <textarea v-model="form.content" placeholder="Content" class="w-full p-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-indigo-300"></textarea>
            </div>

            <input type="file" @change="handleFileUpload" ref="imagefile" class="w-full p-2 mb-5"/>

            <button type="submit" class="bg-indigo-500 text-white px-4 py-2 rounded-md hover:bg-indigo-600">{{ editMode == true ? 'Update': 'Create'}}</button>
            <button v-if="editMode" type="submit" class="bg-red-500 text-white px-4 py-2 rounded-md hover:bg-red-600 ml-2" @click="resetForm">Reset</button>
        </form>
        <div v-for="post in posts.data" :key="post.id" class="mb-4 bg-white p-4 shadow-md rounded-md">
            <h3 class="text-xl font-semibold">{{post.title}}</h3>
            <p class="text-gray-700">{{post.content}}</p>
            <img v-if="post.image" :src="'/storage/'+post.image" class="w-60 h-40 object-cover rounded-lg shadow">

            <button type="button" class="bg-green-500 text-white px-4 py-2 rounded-md hover:bg-green-600 mt-3" @click="editPost(post)">Edit</button>
            <button type="submit" class="bg-red-500 text-white px-4 py-2 rounded-md hover:bg-red-600 ml-2" @click="deletePost(post.id)">Delete</button>
        </div>

        <!-- Pagination -->
        <div v-if="posts.links" class="flex justify-center items-center space-x-2 mt-6">
            <button v-for="(link,index) in posts.links" :key="index" @click="fetchPosts(link.url)" :disabled="!link.url" class="px-4 py-2 rounded-md" :class="{
                'bg-indigo-500 text-white hover:bg-indigo-600' : link.active,
                'bg-gray-500 text-white hover:bg-gray-600' : !link.active && link.url,
                'bg-gray-300 text-gray-600 cursor-not-allowed' : !link.url,
            }" v-html="link.label">
            </button>
        </div>

    </div>
</template>
<script>
export default{
    data(){
        return{
            posts:[],
            form:{
                title:'',
                content:'',
                image:null
            },

            editMode:false,
            editId:null
        }
    },
    methods:{
        async fetchPosts(url="/api/posts"){
            const {data} = await axios.get(url);
            this.posts = data;
        },

        async savePost(){
            const formData = new FormData();
            formData.append('title',this.form.title);
            formData.append('content',this.form.content);

            if(this.form.image){
                formData.append('image',this.form.image);
            }

            if(this.editMode){
                formData.append('_method','PUT');
                await axios.post(`/api/posts/${this.editId}`,formData,{
                   headers:{
                    "Content-Type": "multipart/form-data"
                   } 
                });
            }else{
                await axios.post('/api/posts',this.form,{
                    headers:{
                        'Content-Type':'multipart/form-data'
                    }
                });
            }
            this.resetForm();
            this.fetchPosts();
        },

        editPost(post){
            this.form ={
                title: post.title,
                content: post.content
            };
            this.editId = post.id;
            this.editMode = true;
        },

        resetForm(){
            this.form={
                title:'',
                content:'',
                image:null
            },
            this.$refs.imagefile.value = null;
            this.editId = null;
            this.editMode = false;
        },

        async deletePost(id){
            await axios.delete(`/api/posts/${id}`);
            this.fetchPosts();
        },

        handleFileUpload(event){
            this.form.image = event.target.files[0];
        }

    },
    mounted(){
        this.fetchPosts();
    }
}
</script>
