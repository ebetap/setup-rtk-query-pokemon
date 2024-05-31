Tentu! Berikut saya perpanjang dan perdetail langkah-langkahnya agar lebih lengkap dan terperinci:

### Langkah 1: Persiapan Awal

Pastikan proyek Next.js sudah dibuat dan Tailwind CSS sudah diinstall. Selanjutnya, instal Redux Toolkit dan RTK Query:

```bash
npm install @reduxjs/toolkit @reduxjs/toolkit-query react-redux axios
```

### Langkah 2: Setup Redux Store

Redux Toolkit memungkinkan Anda untuk membuat store Redux dengan cara yang lebih mudah dan intuitif. Buat file `store.js` untuk setup Redux store:

```javascript
// store.js
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query/react';
import { pokemonApi } from './services/pokemon';

// Configure Redux store
export const store = configureStore({
  reducer: {
    [pokemonApi.reducerPath]: pokemonApi.reducer,
    // Add other reducers here if needed
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(pokemonApi.middleware),
});

// Setup listeners for the API
setupListeners(store.dispatch);

export default store;
```

### Langkah 3: Setup RTK Query API

RTK Query menyediakan `createApi` untuk membuat API slice secara deklaratif. Buat file `services/pokemon.js` untuk menentukan API menggunakan `createApi`:

```javascript
// services/pokemon.js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

// Define a service using a base URL and expected endpoints
export const pokemonApi = createApi({
  reducerPath: 'pokemonApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://pokeapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getPokemonById: builder.query({
      query: (id) => `pokemon/${id}`,
    }),
    getAllPokemon: builder.query({
      query: () => 'pokemon',
    }),
    addPokemon: builder.mutation({
      query: (pokemon) => ({
        url: `pokemon`,
        method: 'POST',
        body: pokemon,
      }),
    }),
    updatePokemon: builder.mutation({
      query: ({ id, pokemon }) => ({
        url: `pokemon/${id}`,
        method: 'PUT',
        body: pokemon,
      }),
    }),
    deletePokemon: builder.mutation({
      query: (id) => ({
        url: `pokemon/${id}`,
        method: 'DELETE',
      }),
    }),
  }),
});

// Export hooks for usage in functional components, e.g. useGetPokemonByIdQuery, useAddPokemonMutation, etc.
export const { useGetPokemonByIdQuery, useGetAllPokemonQuery, useAddPokemonMutation, useUpdatePokemonMutation, useDeletePokemonMutation } = pokemonApi;

// Export the API for use in `configureStore` in `store.js`
export { pokemonApi };
```

### Langkah 4: Setup Komponen dan Tampilan

Buat komponen-komponen React untuk menampilkan data Pokemon dan form CRUD:

```javascript
// pages/index.js
import React from 'react';
import { useGetAllPokemonQuery } from '../services/pokemon';

export default function Home() {
  const { data: pokemonList, error, isLoading } = useGetAllPokemonQuery();

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <h1>Pokemon List</h1>
      {pokemonList.map((pokemon) => (
        <div key={pokemon.id}>
          <h2>{pokemon.name}</h2>
          <p>Height: {pokemon.height}</p>
          <p>Weight: {pokemon.weight}</p>
        </div>
      ))}
    </div>
  );
}
```

### Langkah 5: Menggunakan Mutations

Tambahkan form atau penggunaan mutation untuk menambahkan, memperbarui, dan menghapus Pokemon:

```javascript
// pages/add-pokemon.js
import React, { useState } from 'react';
import { useAddPokemonMutation } from '../services/pokemon';

export default function AddPokemon() {
  const [name, setName] = useState('');
  const [height, setHeight] = useState('');
  const [weight, setWeight] = useState('');
  const [addPokemon, { isLoading }] = useAddPokemonMutation();

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await addPokemon({ name, height, weight });
      setName('');
      setHeight('');
      setWeight('');
    } catch (error) {
      console.error('Failed to add pokemon:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" placeholder="Name" value={name} onChange={(e) => setName(e.target.value)} required />
      <input type="text" placeholder="Height" value={height} onChange={(e) => setHeight(e.target.value)} required />
      <input type="text" placeholder="Weight" value={weight} onChange={(e) => setWeight(e.target.value)} required />
      <button type="submit" disabled={isLoading}>
        Add Pokemon
      </button>
    </form>
  );
}
```

### Langkah 6: Menggunakan Router Next.js

Gunakan Next.js router untuk navigasi antar halaman:

```javascript
// pages/_app.js
import '../styles/globals.css';
import { Provider } from 'react-redux';
import { store } from '../store';
import Link from 'next/link';

function MyApp({ Component, pageProps }) {
  return (
    <Provider store={store}>
      <nav>
        <ul>
          <li>
            <Link href="/">
              <a>Home</a>
            </Link>
          </li>
          <li>
            <Link href="/add-pokemon">
              <a>Add Pokemon</a>
            </Link>
          </li>
        </ul>
      </nav>
      <Component {...pageProps} />
    </Provider>
  );
}

export default MyApp;
```

### Langkah 7: Menjalankan Aplikasi

Jalankan aplikasi Next.js dengan perintah:

```bash
npm run dev
```

Aplikasi akan berjalan di `http://localhost:3000`. Anda sekarang dapat mengakses halaman utama dan halaman tambah Pokemon.

### Penjelasan Tambahan

- **Redux Toolkit**: Menggunakan `configureStore` untuk membuat store Redux, dan `setupListeners` untuk mengintegrasikan RTK Query dengan Redux store.
  
- **RTK Query**: Menyediakan hooks seperti `useGetPokemonByIdQuery` dan `useAddPokemonMutation` untuk mendapatkan data dan melakukan operasi mutasi (CRUD) terhadap data Pokemon.

- **Komponen dan Tampilan**: Menggunakan React hooks seperti `useState` untuk form input dan render data Pokemon dengan kondisi loading dan error handling.

- **Router Next.js**: Menggunakan Next.js router dan `Link` untuk navigasi antar halaman.

Dengan mengikuti langkah-langkah di atas, Anda telah memasang Redux Toolkit Query dengan `createApi` pada Next.js app menggunakan JavaScript. Anda dapat menyesuaikan dengan kebutuhan proyek Anda, misalnya menambahkan validasi form, penanganan lebih lanjut terhadap error, atau menyesuaikan tampilan dengan Tailwind CSS.
