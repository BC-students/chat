<template>

  <div>
    
    <button v-if="etatConnexion == false" @click="signInGoogle">Sign in with Google</button>
    <div v-if="etatConnexion == true">
        <button v-if="etatConnexion == true" @click="signOutButton">Log out</button>
        
        
        <!-- <actifs v-if="etatConnexion == true"/> -->

        <button  :class="{newCSS:newGlobal}"  @click="rubrique = 'global'">global <span v-if="newGlobal">new</span></button>
        <button  :class="{newCSS:newPrive}" @click="rubrique = 'prive'">privé <span v-if="newPrive">new</span></button>
        <button  :class="{newCSS:newPerso}" @click="rubrique = 'perso'">perso <span v-if="newPerso">new</span></button>
    </div>
    <listeChat :typeSectionProps='rubrique' :notificationVu="notificationVu" v-if="etatConnexion == true"/>
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
    rubrique: 'global',
    newGlobal: false,
    newPrive: false,
    newPerso: false,
    notificationVu: []
    
    
  }
    
    

},





methods: {

signInGoogle (){
            const provider = new firebase.auth.GoogleAuthProvider()
        firebase.auth().signInWithPopup(provider).then((result) => {
          this.etatConnexion = true
          this.notification()
          this.checkChange ()

          // si new membres
          db.collection('users').doc(firebase.auth().currentUser.uid).get().then(doc => {
              if (doc.exists) {
                  
              } else {
                  //recup user
                  var membresGlobal ={}
                  db.collection("users").get()
                .then(querySnapshot => {
                    querySnapshot.forEach(doc => {
                        membresGlobal[doc.id] = true
                    });
                })


                  //ajout dans global
                  db.collection("sections").where("type", "==", "global")
                  .get()
                  .then(function(querySnapshot) {
                      querySnapshot.forEach(doc => {
                          db.collection("sections").doc(doc.id).update({membres: membresGlobal})
                      });
                  })
                  .catch(function(error) {
                      console.log("Error getting documents: ", error);
                  });
                }
          })





          db.collection('users').doc(firebase.auth().currentUser.uid).set({
            nom: firebase.auth().currentUser.displayName,
            actif: true
          }).then(() => {
            this.notification ()
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

        

        
        
      },

     notification () {

        db.collection("sections")
        .get()
        .then(querySnapshot => {
          var idUser = firebase.auth().currentUser.uid
          this.notificationVu = []
            querySnapshot.forEach(doc => {
              console.log(this.notificationVu)
                if(doc.data().vu[idUser] != true ){
                    var infoSection = [doc.id,doc.data().type]
                     this.notificationVu.push(infoSection)



                }




            });

            console.log("noti",this.notificationVu);
            this.newGlobal = false
            this.newPrive = false
            this.newPerso = false
            // afficher modif
            for (var i = this.notificationVu.length - 1; i >= 0; i--) {
              if(this.notificationVu[i][1] == 'global'){
                this.newGlobal = true
              }
              else if(this.notificationVu[i][1] == 'prive'){
                this.newPrive = true
              }
              else if(this.notificationVu[i][1] == 'perso'){
                this.newPerso = true
              }





            }
              
               
              
              
            





        })
        .catch(function(error) {
            console.log("Error getting documents: ", error);
        });
         },


         //change

         checkChange () {
          console.log('change Function');
          db.collection("sections")
          .onSnapshot(snapshot => {
              snapshot.docChanges.forEach(change => {
                      console.log("change",change);
                      if(change.type === "modified"){
                      console.log('changed');
                      this.notification ()
                    }
                  
                  
              });
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
.newCSS {
  color:orange;
}
</style>
