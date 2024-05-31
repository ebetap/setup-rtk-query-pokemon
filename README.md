Untuk memasang Redux Toolkit Query dengan `createApi` dan melakukan CRUD menggunakan API Pokemon, berikut adalah langkah-langkahnya. Tutorial ini akan menggunakan Next.js dengan JavaScript.

### Langkah 1: Persiapan Awal

Pastikan Anda telah membuat proyek Next.js. Jika belum, Anda dapat membuatnya dengan perintah:

```bash
npx create-next-app@12.0.7 my-nextjs-app
cd my-nextjs-app
```

### Langkah 2: Instalasi Redux Toolkit dan Dependencies

Instal Redux Toolkit dan dependencies yang diperlukan:

```bash
npm install @reduxjs/toolkit react-redux axios
npm install @reduxjs/toolkit/query react-query-hooks
```

### Langkah 3: Membuat API Client

Buat file `api.js` di dalam folder `lib` untuk menangani HTTP request ke API Pokemon:

```javascript
// lib/api.js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

const pokemonApi = createApi({
  reducerPath: 'pokemonApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://pokeapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getPokemons: builder.query({
      query: (limit = 10) => `pokemon?limit=${limit}`,
    }),
    getPokemonById: builder.query({
      query: (id) => `pokemon/${id}`,
    }),
    addPokemon: builder.mutation({
      query: (newPokemon) => ({
        url: 'pokemon',
        method: 'POST',
        body: newPokemon,
      }),
    }),
    updatePokemon: builder.mutation({
      query: ({ id, updatedPokemon }) => ({
        url: `pokemon/${id}`,
        method: 'PUT',
        body: updatedPokemon,
      }),
    }),
    deletePokemon: builder.mutation({
      query: (id) => ({
        url: `pokemon/${id}`,
        method: 'DELETE',
      }),
    }),
  }),
})

export const {
  useGetPokemonsQuery,
  useGetPokemonByIdQuery,
  useAddPokemonMutation,
  useUpdatePokemonMutation,
  useDeletePokemonMutation,
} = pokemonApi
export default pokemonApi
```

### Langkah 4: Konfigurasi Redux Store

Buat dan konfigurasikan Redux store dengan `redux-toolkit` di `store.js`:

```javascript
// store.js
import { configureStore } from '@reduxjs/toolkit'
import pokemonApi from './lib/api'

const store = configureStore({
  reducer: {
    [pokemonApi.reducerPath]: pokemonApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(pokemonApi.middleware),
})

export default store
```

### Langkah 5: Mengintegrasikan dengan Next.js

Integrasikan Redux store dengan Next.js menggunakan `_app.js`:

```javascript
// pages/_app.js
import { Provider } from 'react-redux'
import store from '../store'
import { QueryClient, QueryClientProvider } from 'react-query'
import { ReactQueryDevtools } from 'react-query/devtools'

const queryClient = new QueryClient()

function MyApp({ Component, pageProps }) {
  return (
    <Provider store={store}>
      <QueryClientProvider client={queryClient}>
        <Component {...pageProps} />
        <ReactQueryDevtools initialIsOpen={false} />
      </QueryClientProvider>
    </Provider>
  )
}

export default MyApp
```

### Langkah 6: Menggunakan API di Halaman Next.js

Gunakan hooks yang telah dibuat untuk mengambil data dari API dan melakukan operasi CRUD di halaman Next.js:

```javascript
// pages/index.js
import { useGetPokemonsQuery, useAddPokemonMutation } from '../lib/api'

export default function Home() {
  const { data: pokemons, error, isLoading } = useGetPokemonsQuery()
  const [addPokemon] = useAddPokemonMutation()

  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error: {error.message}</div>

  const handleAddPokemon = async () => {
    const newPokemon = { name: 'Pikachu', type: 'Electric' }
    try {
      await addPokemon(newPokemon)
    } catch (error) {
      console.error('Failed to add:', error)
    }
  }

  return (
    <div>
      <h1>Pokemon List</h1>
      <button onClick={handleAddPokemon}>Add Pokemon</button>
      <ul>
        {pokemons?.results.map((pokemon) => (
          <li key={pokemon.name}>{pokemon.name}</li>
        ))}
      </ul>
    </div>
  )
}
```

### Langkah 7: Menjalankan Aplikasi

Jalankan aplikasi Next.js Anda:

```bash
npm run dev
```

Akses halaman `http://localhost:3000` untuk melihat aplikasi berjalan.

### Kesimpulan

Anda telah membuat aplikasi Next.js yang menggunakan Redux Toolkit Query dengan `createApi` untuk melakukan operasi CRUD pada API Pokemon. Anda juga telah mengintegrasikan Redux Toolkit dengan Next.js untuk manajemen state. Jangan lupa untuk menyesuaikan dengan kebutuhan proyek Anda sendiri dan melakukan penanganan error serta optimisasi tambahan sesuai dengan kebutuhan.
