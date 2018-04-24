<template>

  <div>
    
    <button v-if="etatConnexion == false" @click="signInGoogle">Sign in with Google</button>
    <div v-if="etatConnexion == true">
        <button v-if="etatConnexion == true" @click="signOutButton">deco</button>
        
        
        <!-- <actifs v-if="etatConnexion == true"/> -->

        <button @click="rubrique = 'global'">global</button>
        <button @click="rubrique = 'prive'">privé</button>
        <button @click="rubrique = 'perso'">perso</button>
    </div>
    <listeChat :typeSectionProps='rubrique' v-if="etatConnexion == true"/>
  </div>
 
</template>

<script>

import firebase from 'firebase'
import listeChat from './components/listeChat'
import actifs from './components/actifs'
import { db } from './main'





export default {
  name: 'App',
  components: {
    
    listeChat,
    actifs
    
   
  },

  data () {
  return{
    user: null,
    Connexion: this.etatConnexion,
    rubrique: 'global'
    
    
  }
    
    

},





methods: {

signInGoogle (){
            const provider = new firebase.auth.GoogleAuthProvider()
        firebase.auth().signInWithPopup(provider).then((result) => {
          this.etatConnexion = true
          db.collection('users').doc(firebase.auth().currentUser.uid).set({
            nom: firebase.auth().currentUser.displayName,
            actif: true
          }).then(function() {
            console.log("Document successfully updated!");
      }).catch(function(error) {
    console.error("Error writing document: ", error);
      });
       
        }).catch(err => console.log(error))





          this.userName = firebase.auth().currentUser.displayName

          
          



    },
    signOutButton () {
        // modif actif

          db.collection('users').doc(firebase.auth().currentUser.uid).update({
            actif: false
          }).then(() => {
                  firebase.auth().signOut().then(() => {
                this.etatConnexion = false
              this.user = ''
              }).catch(err => console.log(error))
            console.log("Document successfully updated!");
      }).catch(err => {
    console.error("Error writing document: ", error);
      });

        

        
        
      }

},

created () {
  document.addEventListener('beforeunload', this.signOutButton)
},

updated () {
  this.user = this.userName
  // ajout de user

          
          
}


}




</script>

<style>

</style>
