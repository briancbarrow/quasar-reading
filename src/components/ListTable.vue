<script setup lang="ts">
import { onMounted, ref, computed, Ref, ComputedRef } from 'vue';

type Genre = {
  createdTime: string;
  fields: {
    Name: string;
  };
  id: string;
};

type Book = {
  createdTime: string;
  fields: {
    Author: string;
    Category: string;
    Status: 'In Progress' | 'To Read' | 'Finished' | "Won't Finish";
    Title: string;
  };
  id: string;
};

const books: Ref<Book[]> = ref([]);
const genres: Ref<Genre[]> = ref([]);

function getBooks() {
  fetch('https://api.airtable.com/v0/appV4vhlXofp6BgID/tblQ6ii6JrKorm0jw', {
    headers: {
      Host: 'api.airtable.com',
      'Content-Type': 'application/json',
      Authorization: `Bearer ${process.env.AIRTABLE_KEY}`,
    },
  })
    .then((response) => response.json())
    .then((result) => {
      console.log('books', result);
      books.value = result.records;
    });
}

function getGenres() {
  fetch('https://api.airtable.com/v0/appV4vhlXofp6BgID/tble4uTU3ObmVYNJ4', {
    headers: {
      Host: 'api.airtable.com',
      'Content-Type': 'application/json',
      Authorization: `Bearer ${process.env.AIRTABLE_KEY}`,
    },
  })
    .then((response) => response.json())
    .then((result) => {
      console.log('genres', result);
      genres.value = result.records;
    });
}

type FilteredBooks = {
  [key: string]: Book[];
};

const filteredBooks: ComputedRef<FilteredBooks> = computed(() => {
  const result: FilteredBooks = {};
  genres.value.forEach((genre) => {
    result[genre.fields.Name] = books.value.filter((book) => {
      return book.fields.Category === genre.fields.Name;
    });
  });
  return result;
  //   return author.books.length > 0 ? 'Yes' : 'No'
});

const statusClasses: { [key: string]: string } = {
  'In Progress': 'blue',
  'To Read': 'amber',
  Finished: 'green',
  "Won't Finish": 'red',
};

function editBook(id: string) {
  console.log('edit book', id);
}

const columns = [
  {
    name: 'title',
    required: true,
    label: 'Title',
    align: 'left',
    field: (row) => row.fields.Title,
    format: (val) => `${val}`,
    sortable: true,
  },
  {
    name: 'author',
    align: 'center',
    label: 'Author',
    field: (row) => row.fields.Author,
    sortable: true,
  },
  {
    name: 'status',
    label: 'Status',
    field: (row) => row.fields.Status,
    sortable: true,
  },
  {
    name: 'edit',
    label: 'Edit',
    field: (row) => `<button :click="editBook(${row.id})">Edit</button>`,
  },
];

onMounted(() => {
  getBooks();
  getGenres();
});
</script>

<template>
  <template v-for="genre in genres" :key="genre.fields.Name">
    <div class="q-pa-md">
      <q-table
        flat
        bordered
        :title="genre.fields.Name"
        dense
        :rows="filteredBooks[genre.fields.Name]"
        :columns="columns"
        row-key="name"
        :rows-per-page-options="[0]"
      >
        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td key="title" :props="props">
              {{ props.row.fields.Title }}
            </q-td>
            <q-td key="author" :props="props">
              {{ props.row.fields.Author }}
            </q-td>
            <q-td key="status" :props="props">
              <q-badge :color="statusClasses[props.row.fields.Status]">
                {{ props.row.fields.Status }}
              </q-badge>
            </q-td>
            <q-td key="edit" :props="props">
              <button @click="editBook(props.row.id)">Edit</button>
            </q-td>
          </q-tr>
        </template>
      </q-table>
    </div>
  </template>
</template>
