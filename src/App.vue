<template>
  <div id="app" class="container">
    <br />
    <h6>
      Der verfügbare Speicher für IndexedDB beträgt bei dir ungefähr: <b>{{ totalSpace }} GB</b>
    </h6>
    <br />
    <h2>indexedDB Testsite</h2>
    <div class="d-flex">
      <div class="w-50">
        <h4>My Friends</h4>
        <ul class="list-group mb-4">
          <li
            v-for="(friend, i) of friends"
            :key="i"
            class="list-group-item list-group-item-primary d-flex my-1"
          >
            <span>{{ friend.name.first }} {{ friend.name.last }}</span>
            <span class="flex-grow-1"></span>
            <span @click="addFriend(friend)" class="btn-span"><i class="fas fa-plus-square"></i></span>
          </li>
        </ul>
        <button class="btn btn-primary" @click="addAll()">Store all</button>
      </div>
      <div class="w-50 ms-5">
        <h4>My Friends in the Database</h4>
        <table class="table">
          <thead>
            <th>ID</th>
            <th>First Name</th>
            <th>Last Name</th>
            <th>Age</th>
            <th>Action</th>
          </thead>
          <tbody>
            <tr v-for="friend of storedFriends" :key="friend.id">
              <td>{{ friend.id }}</td>
              <td>{{ friend.name.first }}</td>
              <td>{{ friend.name.last }}</td>
              <td>{{ friend.age }}</td>
              <td>
                <span class="btn-span ms-3" @click="removeFriend(friend)"
                  ><i class="fas fa-trash-alt"></i
                ></span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div class="mt-5 row align-items-center">
      <div class="col-1">
        <span class="align-middle">Search id:</span>
      </div>
      <div class="col-1">
        <input @focus="searchResult = false" v-model.number="id" class="form-control" type="text" />
      </div>
      <div class="col-1">
        <button class="ms-3 btn btn-primary" @click="findFriend(id)">Find!</button>
      </div>
      <div class="col-3">
        <div v-if="searchResult && friend != null">
          <span class="fw-bold">
            Found: {{ friend.name.first }} {{ friend.name.last }}, {{ friend.age }}
          </span>
        </div>
        <span v-else-if="searchResult" class="fw-bold text-danger">Not found</span>
      </div>
    </div>
    <div class="mt-3 row align-items-center">
      <div class="col-1">
        <span>Old Name:</span>
      </div>
      <div class="col-2">
        <input v-model="oldName" class="form-control" type="text" />
      </div>
      <div class="col-1">
        <span>New Name:</span>
      </div>
      <div class="col-2">
        <input v-model="newName" class="form-control" type="text" />
      </div>
      <div class="col-1">
        <button class="btn btn-primary" @click="updateFriend()">Change!</button>
      </div>
    </div>

    <div class="mt-3 row align-items-center">
      <div class="col-1">
        <span>From Age:</span>
      </div>
      <div class="col-2">
        <input v-model="fromAge" class="form-control" type="number" />
      </div>
      <div class="col-1">
        <span>To Age:</span>
      </div>
      <div class="col-2">
        <input v-model="toAge" class="form-control" type="number" />
      </div>
      <div class="col-1">
        <button class="btn btn-primary" @click="getFriendsWithAge()">Filter!</button>
      </div>
      <div class="col-3" v-if="filterResult[0]">
        <ul>
          <li v-for="friend of filterResult" :key="friend.id">
            Found: {{ friend.name.first }} {{ friend.name.last }} {{ friend.age }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { openDB } from 'idb';
export default {
  name: 'App',
  data() {
    return {
      friends: [
        {
          id: 1,
          age: 47,
          name: {
            first: 'Sarah',
            last: 'Bridges',
          },
        },
        {
          id: 2,
          age: 26,
          name: {
            first: 'Madge',
            last: 'Shaw',
          },
        },
        {
          id: 3,
          age: 51,
          name: {
            first: 'Jim',
            last: 'McGee',
          },
        },
        {
          id: 4,
          age: 35,
          name: {
            first: 'Randy',
            last: 'Shields',
          },
        },
        {
          id: 5,
          age: 45,
          name: {
            first: 'Luann',
            last: 'Curtis',
          },
        },
      ],
      db: null,
      id: '',
      searchResult: false,
      friend: null,
      storedFriends: [],
      oldName: 'Bridges',
      newName: 'Cerny',
      estimate: '',
      totalSpace: '',

      fromAge: '',
      toAge: '',
      filterResult: [],
    };
  },
  async created() {
    await this.upgradefriends();
    this.getStoredFriends();
  },
  methods: {
    async getStorage() {
      this.estimate = await navigator.storage.estimate();
      this.totalSpace = (this.estimate.quota / 10 ** 9).toFixed(2);
    },
    async upgradefriends() {
      this.db = await openDB('friendsDB1', 1, {
        upgrade(db) {
          const objectStore = db.createObjectStore('friends', { keyPath: 'id' });
          objectStore.createIndex('lastname', 'name.last', { unique: true });
          objectStore.createIndex('age', 'age', { unique: false });
        },
      });
    },
    async getStoredFriends() {
      this.storedFriends = await this.db.getAll('friends');
    },
    async addFriend(friend) {
      await this.db.put('friends', friend);
      this.getStoredFriends();
    },
    async addAll() {
      await this.friends.forEach((element) => {
        this.db.put('friends', element);
      });
      this.getStoredFriends();
    },
    async removeFriend(friend) {
      this.storedFriends = await this.db.delete('friends', friend.id);
      this.getStoredFriends();
    },
    async findFriend(id) {
      this.searchResult = id;
      this.friend = await this.db.get('friends', id);
      // console.log(this.searchResult);
      // console.log(this.friend);
    },
    async updateFriend() {
      const tx = this.db.transaction('friends', 'readwrite');
      const index = tx.store.index('lastname');
      const obj = await index.get(this.oldName);
      obj.name.last = this.newName;
      await this.db.put('friends', obj);
      await tx.done;

      this.getStoredFriends();
    },
    async getFriendsWithAge() {
      this.filterResult = await this.db.getAllFromIndex(
        'friends',
        'age',
        IDBKeyRange.bound(Number(this.fromAge), Number(this.toAge)),
      );
    },
  },
};
</script>

<style>
.btn-span {
  cursor: pointer;
}
</style>
