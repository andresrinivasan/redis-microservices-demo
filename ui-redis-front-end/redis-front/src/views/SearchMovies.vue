<template>
  <div id="app" class="px-8 py-8 min-h-screen">

    <h1> <b-icon-camera-video/>  Search Movies with RediSearch</h1>
    
    <b-form @submit="onSubmit" @reset="onReset" >
      <b-input-group
        >
        <b-form-input
            v-model="searchQuery"
        >
        </b-form-input>
        <b-input-group-append>
          <b-button size="sm" text="Button" @click="search"  >Search</b-button>
        </b-input-group-append>
      </b-input-group>
    </b-form>

<div class="mb-2 mt-2 text-justify">
      {{ searchResult.meta.totalResults }} movies found in {{ searchResult.meta.elaspedTimeMs }} ms
      <span v-if="searchResult.meta.totalResults > 0" >
      , showing {{ searchResult.meta.offset + 1 }} to {{ searchResult.meta.limit }}
      </span>
</div>

<b-row>
  <div 
    v-for="doc in searchResult.docs"
    :key="doc.movie_id"
    class="col-md-3 col-4 my-1">

    <b-card>
      <b-card-title>{{doc.body.title}}</b-card-title>
       <b-card-sub-title class="mb-2">{{doc.body.release_year}} </b-card-sub-title>
      <b-card-text>
        {{ doc.body.plot }}
      </b-card-text>

     <b-card-text>
       <img width=130 :src="doc.body.poster" />
     </b-card-text>

      <template v-slot:footer >
        <b-row class="small">
        <b-col>
        {{ doc.body.genre }}
        </b-col>
        <b-col>
          <b-button size="sm" @click="goToMovie( doc.body.movie_id)">View</b-button>
        </b-col>
        <b-col class="text-right">
        {{ Number.parseFloat(doc.body.rating).toFixed(1) }}
        </b-col>
        </b-row>
      </template>

    </b-card>


  </div>
</b-row>
  

  </div>
</template>

<script>
import { RepositoryFactory } from './../repositories/RepositoryFactory'
const RediSearchRepository = RepositoryFactory.get('dataStreamsRedisHashSyncRepository');


export default {
  name: "Search",
  data() {
    return {
      searchQuery : "*",
      searchResult : {},
    }
  },
  created() {
    this.$parent.contextualHelp = 
        "This search form is based on RediSearch on the Movie index (<i>\"idx:ms:search:index:movies\"</<i>).<br>You can use various query such as:"+
        "<ul><li>wars -CIVIL</li></ul>"+
        "<p>Update, create a movie and you will see that the index is updated. (If the <a href='/services'>service</a> is running)</p>";

    this.search();

  },
  methods : {
    onSubmit(evt) {
      evt.preventDefault()
      this.search();
    },
    onReset(evt) {
        evt.preventDefault()

    },
    async search() {
      console.log( this.searchQuery );

      const {data} = await RediSearchRepository.search("movies", this.searchQuery);
      console.log(data);
      this.searchResult = data;

    },
    goToMovie(id) {
      console.log(id);
      this.$router.push({ name: 'MovieForm', params: { id: id }});
    }
  }
};
</script>

