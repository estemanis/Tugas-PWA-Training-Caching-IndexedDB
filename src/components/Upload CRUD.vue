<template>
    <div style="margin: auto">
        <div>
            <span>Pilih Gambar Dari File</span>
        </div>

        <div v-if="imageSource != null && imageShow">
            <div>
                <img :src="imageSource" style="width: 300px; height: 200px;"/>
            </div>

            <div>
                <button @click="deleteImage" style="background-color: red; margin-top: 10px; margin-bottom: 10px; color: white;">DELETE</button>
            </div>
        </div>

        <div>
            <div>
                <input ref="imageInput" type="file" accept="image/*" @change="onImageInput"/>
            </div>

            <div v-if="addedImage">
                <button @click="readImage" style="background-color: aqua; margin-top: 10px; margin-bottom: 10px;">Show Image</button>
            </div>
        </div>
    </div>
</template>

<script setup>
import {onMounted, ref} from 'vue';

// references
const imageInput = ref(null);

const dbName = "image_upload";
const objectStoreName = "uploaded_images";
const uploaded_images = ref([]);

// Image File CRUD
const imageShow = ref(false);
const imageStored = ref(false);
const imageSource = ref(null);
const addedImage = ref(false);
const updatedData = ref(null);
const deletedData = ref(null);

const onImageInput = (e) => {
    const selectedImage = e.target.files[0];

    if (!imageStored.value) {
        addImage(selectedImage);
    } else {
        updateImage(selectedImage);
    }
}

const addImage = (image) => {
    const newObjectUrl = URL.createObjectURL(image);
    const imageToAdd = { "id": 1, "file": newObjectUrl };
    console.log("ADD: " + newObjectUrl);

  const request = indexedDB.open(dbName, 5);

  request.onerror = (event) => {
    console.error("Error opening database:", event.target.error);
  };

  request.onupgradeneeded = (event) => {
    const db = event.target.result;

    const objectStore = db.createObjectStore("uploaded_images");
    objectStore.createIndex("image", "image", { unique: false });
  };

  request.onsuccess = (event) => {
    const db = event.target.result;

    const addTransaction = db.transaction("uploaded_images", "readwrite");
    const customerObjectStore = addTransaction.objectStore("uploaded_images");

    const addRequest = customerObjectStore.add(imageToAdd);

    addRequest.onsuccess = (event) => {
      console.log("Data added successfully");
      addedImage.value = "Yes";
    };

    addRequest.onerror = (event) => {
      console.error("Error adding data", event.target.error);
      addedImage.value = "No";
    };

    addTransaction.oncomplete = () => {
      console.log("Add transaction completed");
      db.close();
    };
  };
}

const readImage = () => {
    console.log("Reading Stored Images");

    const request = indexedDB.open(dbName, 5);

  request.onerror = (event) => {
    console.error("Error opening database:", event.target.error);
  };

  request.onsuccess = (event) => {
    const db = event.target.result;

    const readTransaction = db.transaction("uploaded_images", "readonly");
    const customerObjectStore = readTransaction.objectStore("uploaded_images");

    const customersCursor = customerObjectStore.openCursor();

    customersCursor.onsuccess = (event) => {
      const cursor = event.target.result;

      if (cursor) {
        console.log('image.value:', cursor.value)
        uploaded_images.value.push(cursor.value);
        const storedImage = cursor.value;
        imageSource.value = storedImage.file;
        imageStored.value = true;
        imageShow.value = true;
        cursor.continue();
      } else {
        console.log("Data read successfully");
        db.close();
      }
    };

    customersCursor.onerror = (event) => {
      console.error("Error reading data", event.target.error);
      db.close();
    };
  };
}

const updateImage = (image) => {
    const updateObjectUrl = URL.createObjectURL(image);
    console.log("UPDATE: " + updateObjectUrl);

    const request = indexedDB.open(dbName, 5);

  request.onerror = (event) => {
    console.error("Error opening database:", event.target.error);
  };

  request.onsuccess = (event) => {
    const db = event.target.result;
    const updatedImage = {"id": 1, "file": updateObjectUrl};
    const updateTransaction = db.transaction("uploaded_images", "readwrite");
    const customerObjectStore = updateTransaction.objectStore("uploaded_images");

    const updateRequest = customerObjectStore.put(updatedImage);

    updateRequest.onsuccess = (event) => {
      console.log("Data updated successfully");
      updatedData.value = "Yes";
      imageShow.value = false;
    };

    updateRequest.onerror = (event) => {
      console.error("Error updating data", event.target.error);
      updatedData.value = "No";
    };

    updateTransaction.oncomplete = () => {
      console.log("Update transaction completed");
      db.close();
    };
  };    
}

const deleteImage = () => {
    const request = indexedDB.open(dbName, 5);
    const keyToDelete = "1";
    request.onerror = (event) => {
    console.error("Error opening database:", event.target.error);
    };

    request.onsuccess = (event) => {
        const db = event.target.result;

        const deleteTransaction = db.transaction("uploaded_images", "readwrite");
        const deleteObjectStore = deleteTransaction.objectStore("uploaded_images");

        const deleteRequest = deleteObjectStore.delete(keyToDelete);

        deleteRequest.onsuccess = (event) => {
            console.log("Data deleted successfully");
            deletedData.value = "Yes";
            imageSource.value = null;
            imageStored.value = false;
            imageShow.value = false;
            imageInput.value.value = null;
        };

        deleteRequest.onerror = (event) => {
            console.error("Error deleting data", event.target.error);
            deletedData.value = "No";
        };

        deleteTransaction.oncomplete = () => {
            console.log("Delete transaction completed");
            db.close();
        };
    };
}

onMounted(() => {
    deleteImage();
})
</script>
