<template>

  <div>
    <!-- //pop up nouvelle section-->
      <div v-if="showModal == true" id="newSection">
          <label for="titre">Nom du channel</label>
          <input v-model="titreSection" name="nomSection" type="text">
          <button @click="newSection">Créer</button>
          <button @click="showModal = false">Annuler</button>
      </div>
    <!-- fin pop up -->
    <!-- //pop up  ajouter un membre-->
      <div class="popupCSS" v-if="showModal2 == true" id="newMembre">
          <label for="nom">ajouter un mebre</label>
          <input v-model="nomMembre" name="nomMembre" type="text">
          <button v-if="ajouterId != ''" @click="ajouterMembre">Ajouter</button>
          <button @click="showModal2 = false">Annuler</button>
          <!-- liste déroulante -->
          <div v-for="membre in filteredMembres">
           <span @click="nomMembre = membre[1]; ajouterId = membre[0]">{{membre[1]}}</span>
          </div>
      </div>
    <!-- fin pop up -->

    <ul class="listeChatCSS">
      <li class="listeSectionsCSS" v-for="section in listeSections" @click="sectionProps=section[0]; messagePerso(section[0])">
        {{section[1]}}  
        <button v-if="typeSection == 'prive'" @click="showModal2 = true; showModal2Value = section[0]">+</button> 
        <button v-if="typeSectionProps == 'prive'" @click="supprimerSection(section[0])">X</button>
      </li>
      <li v-if="typeSectionProps != 'perso'"><button @click="showModal = true">+</button></li>
    </ul>
    <chatBox v-if="sectionProps != ''" class="tchatZone" :sectionActive='sectionProps'  id="centrerCSS"/>
  </div>
 
</template>

<script>

import firebase from 'firebase'
import chatBox from './chatBox'
import { db } from '../main'





export default {
  name: 'listeChat',
  components: {
    
    chatBox
    
    
   
  },
  props: ['typeSectionProps'],

 

  data () {
  return{
    user: null,
    listeSections: [],
    titreSection: '',
    showModal: false,
    showModal2: false,
    showModal2Value: null,
    sectionProps : '',
    typeSection : 'global',
    nomMembre:'',
    membres: ['moi'],
    ajouterId: ''
  }
    
  
    
    

},





methods: {

  afficherSections () {
      //privé
      if(this.typeSectionProps == 'prive'){
        var idUser = firebase.auth().currentUser.uid
        var where = 'membres.' + idUser
        db.collection('sections').where('type', '==', this.typeSectionProps).where(where, '==', true).onSnapshot(querySnapshot => {
        this.listeSections = []
        querySnapshot.forEach(doc => {

           var infoMsg = [doc.id,doc.data().nom]
          this.listeSections.push(infoMsg)
        })

  })

      }
      //prive
      else if (this.typeSectionProps == 'perso') {
         db.collection('users').onSnapshot(querySnapshot => {
        this.listeSections = []
        querySnapshot.forEach(doc => {
          if(doc.data().nom == firebase.auth().currentUser.displayName){} 
           else {var infoMsg = [doc.id,doc.data().nom]
          this.listeSections.push(infoMsg)}
        })

      })
      }


      //global
      else {
      db.collection('sections').where('type', '==', this.typeSectionProps).onSnapshot(querySnapshot => {
        this.listeSections = []
        querySnapshot.forEach(doc => {

           var infoMsg = [doc.id,doc.data().nom]
          this.listeSections.push(infoMsg)
        })

  })
    }
},

  newSection () {
      this.showModal = false
      //prive
      if(this.typeSectionProps == 'prive'){
        var idUser = firebase.auth().currentUser.uid
        var membres = {}
        membres[idUser] = true
          db.collection("sections").add({
      nom: this.titreSection,
      type: this.typeSectionProps,
      membres: membres
      })
      .then(function(docRef) {
          console.log("Document written with ID: ", docRef.id);
          //ajout membre
          // db.collection('sections/' + docRef.id + '/membres').add({
          //   dateAjout: new Date,
          //   })
            // .then(function(docRef) {
            //     console.log("Document written with ID: ", docRef.id);
            // })
            // .catch(function(error) {
            //     console.error("Error adding document: ", error);
            // });

      })
      .catch(function(error) {
          console.error("Error adding document: ", error);
      });

      




      }
      else {
      //global
      db.collection("sections").add({
    nom: this.titreSection,
    type: this.typeSectionProps,
    })
    .then(function(docRef) {
        console.log("Document written with ID: ", docRef.id);
    })
    .catch(function(error) {
        console.error("Error adding document: ", error);
    });
  }
  },

  supprimerSection(index){
    db.collection("sections").doc(index).delete().then(function() {
    console.log("Document successfully deleted!");
}).catch(function(error) {
    console.error("Error removing document: ", error);
});
  },

  switchSection(index){
    console.log(index);
  },

  ajouterMembre() {
    var sectionProps = this.sectionProps;
    var membresSection


    var docRef = db.collection("sections").doc(sectionProps);

      docRef.get().then(doc => {
          if (doc.exists) {
              console.log("Document data:", doc.data().membres);
              membresSection = doc.data().membres
              membresSection[this.ajouterId] = true
              console.log(membresSection);
              docRef.update({
                membres: membresSection
                })
                .then(function(docRef) {
                    console.log("Document written with ID: ", docRef.id);
                })
                .catch(function(error) {
                    console.error("Error adding document: ", error);
                });
          } else {
              // doc.data() will be undefined in this case
              console.log("No such document!");
          }
      }).catch(function(error) {
          console.log("Error getting document:", error);
      });





  },

  recuperationUser() {
    db.collection("users").onSnapshot(querySnapshot => {
        this.membres = []
        var membresListe = []
        querySnapshot.forEach(doc => {
            var infoMembre = [doc.id, doc.data().nom]
            this.membres.push(infoMembre)
        });

        console.log("liste membres ", membresListe.join(", "));
        // this.membres = membresListe
    });
  },

  messagePerso (id) {
    if(this.typeSectionProps == 'perso'){
    var currentUserId = firebase.auth().currentUser.uid

    var idDoc = id + currentUserId
    var idDoc2 = currentUserId + id 

    var docRef = db.collection("sections");

      db.collection("sections").doc(idDoc).get().then(doc => {
          if (doc.exists) {
              console.log("Document data:", doc.data());
              this.sectionProps = idDoc
          } else {
              // doc.data() will be undefined in this case
              console.log("No such document!");

                   db.collection("sections").doc(idDoc2).get().then(doc => {
                      if (doc.exists) {
                      console.log("Document data:", doc.data());
                      this.sectionProps = idDoc2
                     } else {
                        
                    console.log("No such document!");

                     db.collection("sections").doc(idDoc).set({
                        type: "perso",
                    })
                    .then(docRef => {
                        console.log("Document written with ID: ", docRef.id);
                        this.sectionProps = idDoc
                    })
                    .catch(error => {
                        console.error("Error adding document: ", error);
                    });




                      





          }
      }).catch(function(error) {
          console.log("Error getting document:", error);
      });




          }
      }).catch(function(error) {
          console.log("Error getting document:", error);
      });
  }


}
 
        
      

},

created () {
  this.afficherSections()
  this.recuperationUser()
},

watch: {
  typeSectionProps: function() {
          this.typeSection = this.typeSectionProps;
          this.afficherSections ()

        }
    
  
    
},

computed:
{
    filteredMembres:function()
    {   
       var self=this;
       return this.membres.filter(function(cust){return cust[1].toLowerCase().indexOf(self.nomMembre.toLowerCase())>=0;});
       //return this.membres;
    }
}


}




</script>

<style>
 .tchatZone{
  margin:auto;
 }
 #centrerCSS {
  margin:auto;
  margin-top: 10px;
  width: 500px;
 }

 .listeSectionsCSS{
  cursor: pointer
 }

 .listeChatCSS {
  position: absolute;
  top: 30px;
 }

 .popupCSS {
  position: absolute;
  margin:auto;
  background-color: #e8d6c9;
  z-index: 1000;
  padding: 20px;
 }
</style>
