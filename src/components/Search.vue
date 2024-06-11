<template>
<div>
    <h1>Search via Snomed CT</h1>
    <input type="text" v-model="search" @input="handleInput">
    <div v-if="searchResults.length" class="tab-div">
        <div v-for="description in descriptionsArr" :key="description">
            <button class="tab" @click="replaceSearch(description)"> {{ description }}</button>
        </div>
    </div>
    <div v-else>
        No results found.
    </div>
</div>
</template>

<script>
import debounce from 'lodash/debounce';

export default {
    data() {
        return {
            search: "",
            searchResults: [],
            lastSearchTerm: "",
            firstConceptId: null,
            descriptionsArr: []
        }
    },
    methods: {
        handleInput: debounce(function () {
            this.performSearch();
        }, 500),
        async performSearch() {
            const currentSearchTerm = this.search.trim();
            if (currentSearchTerm === this.lastSearchTerm) {
        
                return;
            }
            this.descriptionsArr = [];

            try {
                const res = await fetch(`https://snowstorm.snomedtools.org/snowstorm/snomed-ct/multisearch/descriptions?term=${currentSearchTerm}&active=true&conceptActive=true&contentScope=ALL_PUBLISHED_CONTENT&offset=0&limit=10`)
                const data = await res.json();
                console.log(data)
                this.searchResults = data.items;
                this.lastSearchTerm = currentSearchTerm;
                if (this.searchResults.length > 0) {
                    // store the concept ID 
                    console.log("hello")
                    this.firstConceptId = this.searchResults[0].concept.conceptId;
                    console.log(this.firstConceptId)
                    console.log("id")
                } else {
                    this.firstConceptId = null;
                }
            } catch (error) {
                console.log(error)
            }
            try {
                const conceptRes = await fetch(`https://snowstorm.snomedtools.org/snowstorm/snomed-ct/browser/MAIN/concepts/${this.firstConceptId}`)
                const data = await conceptRes.json();
                let des = data.descriptions
                for (let i = 0; i < des.length; i++) {
                if(this.descriptionsArr.indexOf(des[i])===-1)
                    this.descriptionsArr.push(des[i].term)
                    console.log(this.descriptionsArr)
                }
            } catch (error) {
                console.log(error)
            }
        },
        replaceSearch(description){
            this.search = description;
            this.performSearch();
        }
    }
}
</script>

<style>
.tab-div{
    display: flex;
    justify-content: center;
    margin-top: 1rem;
}
.tab {
    background: gray;
    padding: 0.5rem 1rem;
    color: white;
    border-radius: 2rem;
    margin: 0.3rem;
    border: none;
}

/* <!-- multisearch --> */
/* https://snowstorm.snomedtools.org/snowstorm/snomed-ct/multisearch/descriptions?term=fever&active=true&conceptActive=true&contentScope=ALL_PUBLISHED_CONTENT&offset=0&limit=10 */

/* <!-- seach by concept id --> */

/* https://snowstorm.snomedtools.org/snowstorm/snomed-ct/browser/MAIN/concepts/386661006 */
</style>
