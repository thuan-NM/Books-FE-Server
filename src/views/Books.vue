<script setup>
import { onMounted, ref, watchEffect } from 'vue';
import BooksOverview from './BooksOverview.vue';
import booksService from '@/services/books.service';
import AuthorService from '@/services/publisher.service';
import {
  Dialog,
  DialogOverlay,
  DialogPanel,
  TransitionChild,
  TransitionRoot,
} from '@headlessui/vue';
import { message } from 'antd';
const products = ref([]);
const selectedBook = ref(null);
const isModalOpen = ref(false);
const newBook = ref({});
const isAddBookModalOpen = ref(false);
watchEffect(async () => {
  // Check if filters is not null before making the API call
  {
    const allProducts = await booksService.getAll();
    products.value = allProducts.data.map(product => ({
      id: product._id,
      name: product.name,
      bookPicture: product.bookPicture,
      pubyear: product.pubyear,
      quantity: product.quantity,
      price: product.price,
      author: product.author,
      description: product.description,
    }));
  }
});

function openModal(book) {
  selectedBook.value = book;
  isModalOpen.value = true;
}

function closeModal() {
  isModalOpen.value = false;
}

const isEditModalOpen = ref(false);
const confirmDeleteModal = ref(false);
const allAuthor = ref([]);
const isLoading = ref(true);

onMounted(async () => {
  const response = await AuthorService.getAll();
  allAuthor.value = response.data;
  let author = {}
  for (author of allAuthor.value) {
    console.log(author.name);
  }
  // Kiểm tra dữ liệu từ API
  isLoading.value = false
});

function openEditModal(book) {
  selectedBook.value = book;
  selectedBook.value.authorId = book.author ? book.author._id : null;
  if (!selectedBook.value.author || !selectedBook.value.author.name) {
    console.error('selectedBook.author.name does not exist');
  }
  if (!allAuthor.value) {
    console.error('allAuthor.value does not exist');
  }
  isEditModalOpen.value = true;
}


function closeEditModal() {
  isEditModalOpen.value = false;
}

function openConfirmDeleteModal(book) {
  selectedBook.value = book;
  confirmDeleteModal.value = true;
}

function closeConfirmDeleteModal() {
  confirmDeleteModal.value = false;
}

const saveProduct = async () => {
  // Lưu thông tin sản phẩm
  selectedBook.value.price = Number(selectedBook.value.price)
  const res = await booksService.update(selectedBook.value.id, selectedBook.value);
  if (res.document.isSuccess == true) {
    // Làm mới danh sách sản phẩm
    const allProducts = await booksService.getAll();
    products.value = allProducts.data.map(product => ({
      id: product._id,
      name: product.name,
      bookPicture: product.bookPicture,
      pubyear: product.pubyear,
      quantity: product.quantity,
      price: Number(product.price),
      author: product.author,
      description: product.description,
    }));
    message.success(res.document.message)
    closeEditModal();
  } else {
    message.error(res.document.message)
  }
};


const deleteProduct = async () => {
  const res = await booksService.delete(selectedBook.value.id);
  // Refresh the product list
  if (res.document.isSuccess == true) {
    const allProducts = await booksService.getAll();
    products.value = allProducts.data.map(product => ({
      id: product._id,
      name: product.name,
      bookPicture: product.bookPicture,
      pubyear: product.pubyear,
      quantity: product.quantity,
      price: product.price,
      author: product.author,
      description: product.description,
    }));
    message.success(res.document.message)
    closeConfirmDeleteModal();
  }
  else{
    message.error(res.document.message)
  }
};

function openAddBookModal() {
  isAddBookModalOpen.value = true;
}

function closeAddBookModal() {
  isAddBookModalOpen.value = false;
}

const addBook = async () => {
  const res = await booksService.create(newBook.value);
  // Refresh the book list


  if (res.document.isSuccess == true) {
    const allProducts = await booksService.getAll();
    products.value = allProducts.data.map(product => ({
      id: product._id,
      name: product.name,
      bookPicture: product.bookPicture,
      pubyear: product.pubyear,
      quantity: product.quantity,
      price: product.price,
      author: product.author,
      description: product.description,
    }));
    message.success(res.document.message)
    closeModal();
  } else {
    message.error(res.document.message)
  }
  closeAddBookModal();
};
</script>

<template>
  <div class="bg-white">
    <div class="mx-auto max-w-10xl px-4 py-16 sm:px-6 sm:py-6 lg:max-w-10xl lg:px-8">
      <div class="mt-6 grid grid-cols-1 pl-auto gap-x-6 gap-y-16 sm:grid-cols-2 lg:grid-cols-4 xl:gap-x-0">
        <div v-for="product in products" :key="product.id" @click="openModal(product)"
          class="group relative h-5/6 w-5/6 mx-auto">
          <div
            class="aspect-h-1 aspect-w-1 w-full overflow-hidden rounded-md bg-gray-200 lg:aspect-none group-hover:opacity-75 lg:h-full">
            <img :src="product.bookPicture" class="h-full w-full object-cover object-center lg:h-full lg:w-full" />
          </div>
          <div class="mt-4 flex justify-between">
            <div>
              <h3 class="text-sm text-gray-500">
                <a :href="product.href">
                  <span aria-hidden="true" class="absolute inset-0 truncate"></span>
                  {{ product.name }}
                </a>
              </h3>
              <p class="mt-1 text-sm text-gray-700">{{ product.author.name }}</p>
            </div>
          </div>
          <div class="flex justify-around mt-4 border-b-2 border-black border-x-2 rounded-full p-2">
            <button class="p-2 border-2 border-gray-400 rounded-full" @click.stop="openEditModal(product)"><i
                class="fa-solid fa-pen-to-square text-xl w-6"></i></button>
            <button class="p-2 border-2 border-gray-400 rounded-full" @click.stop="openConfirmDeleteModal(product)"><i
                class="fa-solid fa-trash text-xl w-6"></i></button>
          </div>
        </div>
        <button
          class="group relative w-5/6 mx-auto bg-transparent p-10 rounded-lg m-auto border-2 border-gray-300 h-full hover:border-4 :hover:border-gray-500 ease-in-out duration-100"
          @click="openAddBookModal"><i class="fa-solid fa-plus text-6xl text-gray-400"></i>
        </button>
      </div>
    </div>
    <BooksOverview v-if="isModalOpen" :book="selectedBook" @close="isModalOpen = false" />
  </div>
  <TransitionRoot as="template" :show="isEditModalOpen" v-if="!isLoading">
    <Dialog as="div" class="fixed z-20 inset-0 overflow-y-auto" @close="closeEditModal">
      <div class="flex items-center justify-center min-h-screen">
        <DialogOverlay class="fixed inset-0 bg-black opacity-30" />
        <TransitionChild as="template" enter="ease-out duration-300" enter-from="opacity-0 scale-95"
          enter-to="opacity-100 scale-100" leave="ease-in duration-200" leave-from="opacity-100 scale-100"
          leave-to="opacity-0 scale-95">
          <DialogPanel class="bg-white rounded-lg p-6 max-w-md mx-auto z-10 w-96">
            <form class="space-y-6">
              <div>
                <label for="name" class="block text-sm font-medium text-gray-700">Book name</label>
                <input id="name" type="text" v-model="selectedBook.name"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
              </div>
              <div>
                <label for="author" class="block text-sm font-medium text-gray-700">Author name</label>
                <select id="author" v-model="selectedBook.author"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
                  <option v-for="authorItem in allAuthor" :key="authorItem._id" :value="authorItem">{{
                    authorItem.name }}</option>
                </select>
              </div>
              <div>
                <label for="address" class="block text-sm font-medium text-gray-700">Quantity</label>
                <input id="address" type="text" v-model="selectedBook.quantity"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
              </div>
              <div>
                <label for="price" class="block text-sm font-medium text-gray-700">Price</label>
                <input id="price" type="text" v-model="selectedBook.price"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
              </div>
              <div>
                <label for="description" class="block text-sm font-medium text-gray-700">Description</label>
                <input id="description" type="text" v-model="selectedBook.description"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
              </div>
              <div class="flex justify-end space-x-2">
                <button type="button" class="px-4 py-2 bg-blue-600 text-white rounded-md"
                  @click="saveProduct">Save</button>
                <button type="button" class="px-4 py-2 bg-gray-200 rounded-md" @click="closeEditModal">Cancel</button>
              </div>
            </form>
          </DialogPanel>
        </TransitionChild>
      </div>
    </Dialog>
  </TransitionRoot>
  <TransitionRoot as="template" :show="confirmDeleteModal">
    <Dialog as="div" class="fixed z-10 inset-0 overflow-y-auto" @close="closeConfirmDeleteModal">
      <div class="flex items-center justify-center min-h-screen">
        <DialogOverlay class="fixed inset-0 bg-black opacity-30" />
        <TransitionChild as="template" enter="ease-out duration-300" enter-from="opacity-0 scale-95"
          enter-to="opacity-100 scale-100" leave="ease-in duration-200" leave-from="opacity-100 scale-100"
          leave-to="opacity-0 scale-95">
          <DialogPanel class="bg-white rounded-lg p-6 max-w-md mx-auto z-10">
            <div class="space-y-4">
              <h2 class="text-xl font-bold text-gray-900">Confirm Delete</h2>
              <p>Are you sure to delete this publisher?</p>
              <div class="flex justify-end space-x-2">
                <button type="button" class="px-4 py-2 bg-red-600 text-white rounded-md"
                  @click="deleteProduct">Delete</button>
                <button type="button" class="px-4 py-2 bg-gray-200 rounded-md"
                  @click="closeConfirmDeleteModal">Cancel</button>
              </div>
            </div>
          </DialogPanel>
        </TransitionChild>
      </div>
    </Dialog>
  </TransitionRoot>
  <TransitionRoot as="template" :show="isAddBookModalOpen">
    <Dialog as="div" class="fixed z-20 inset-0 overflow-y-auto" @close="closeAddBookModal">
      <div class="flex items-center justify-center min-h-screen">
        <DialogOverlay class="fixed inset-0 bg-black opacity-30" />
        <TransitionChild as="template" enter="ease-out duration-300" enter-from="opacity-0 scale-95"
          enter-to="opacity-100 scale-100" leave="ease-in duration-200" leave-from="opacity-100 scale-100"
          leave-to="opacity-0 scale-95">
          <DialogPanel class="bg-white rounded-lg p-6 max-w-md mx-auto z-10 w-96">
            <form class="space-y-6" @submit.prevent="addBook">
              <div>
                <label for="name" class="block text-sm font-medium text-gray-700">Book name</label>
                <input id="name" type="text" v-model="newBook.name"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
              </div>
              <div>
                <label for="bookPicture" class="block text-sm font-medium text-gray-700">Book Picture URL</label>
                <input id="bookPicture" type="text" v-model="newBook.bookPicture"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
              </div>
              <div>
                <label for="pubyear" class="block text-sm font-medium text-gray-700">Publication Year</label>
                <input id="pubyear" type="text" v-model="newBook.pubyear"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
              </div>
              <div>
                <label for="quantity" class="block text-sm font-medium text-gray-700">Quantity</label>
                <input id="quantity" type="number" v-model="newBook.quantity"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
              </div>
              <div>
                <label for="price" class="block text-sm font-medium text-gray-700">Price</label>
                <input id="price" type="number" v-model="newBook.price"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
              </div>
              <div>
                <label for="author" class="block text-sm font-medium text-gray-700">Author name</label>
                <select id="author" v-model="newBook.author"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2">
                  <option v-for="authorItem in allAuthor" :key="authorItem" :value="authorItem">{{
                    authorItem.name }}</option>
                </select>
              </div>
              <div>
                <label for="description" class="block text-sm font-medium text-gray-700">Description</label>
                <textarea id="description" v-model="newBook.description"
                  class="mt-1 block w-full border-gray-300 rounded-md shadow-sm p-2"></textarea>
              </div>
              <div class="flex justify-end space-x-2">
                <button type="submit" class="px-4 py-2 bg-blue-600 text-white rounded-md">Add Book</button>
                <button type="button" class="px-4 py-2 bg-gray-200 rounded-md"
                  @click="closeAddBookModal">Cancel</button>
              </div>
            </form>
          </DialogPanel>
        </TransitionChild>
      </div>
    </Dialog>
  </TransitionRoot>
</template>
