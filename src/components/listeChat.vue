<template>

  <div>
    <!-- //pop up nouvelle section-->
      <b-modal hide-footer v-if="showModal == true" id="newSection" title="Nom du channel">
          <input class="d-block" v-model="titreSection" name="nomSection" type="text">
          <b-btn @click="showModal = false" variant="outline-danger">Annuler</b-btn>
          <b-btn @click="newSection">Créer</b-btn>
          
      </b-modal>
    <!-- fin pop up -->
    <!-- //pop up  ajouter un membre-->
      <b-modal hide-footer v-if="showModal2 == true" id="newMembre" title="Ajouter un membre">
          <img src="../assets/icons/search.png"><input v-model="nomMembre" name="nomMembre" type="text">
          <b-btn v-if="ajouterId != ''" @click="ajouterMembre">Ajouter</b-btn>
          <b-btn @click="showModal2 = false">Annuler</b-btn>
          <!-- liste déroulante -->
          <div v-for="membre in filteredMembres">
           <span @click="nomMembre = membre[1]; ajouterId = membre[0]">{{membre[1]}}</span>
          </div>
      </b-modal>
    <!-- fin pop up -->
    <div id="titreSection">{{nomSection}}</div>
    <b-container id="bodyListeChat" fluid class="w-100">
      <b-row>
        <b-col cols="3">
          <div id="titreTypeSection">{{typeSectionProps}}</div>
          <div>barre de recherhe - comming soon</div>
          <ul class="listeChatCSS">
            <li v-b-modal.newSection id="ajouterSectionCSS" class="listeSectionsCSS" @click="showModal = true" v-if="typeSectionProps != 'perso'"><span style="margin-left:5px;margin-right:5px;">Ajouter une section</span><img src="../assets/icons/add.png" alt=""></li>
            <li :class="{newCSS2:section[2],selection:sectionProps == section[0]}" class="listeSectionsCSS" v-for="section in listeSections" @click="sectionProps=section[0]; messagePerso(section[0]);nomSection = section[1]" >
              {{section[1]}} <span v-if="section[2]">new</span>  
              <button v-b-modal.newMembre v-if="typeSection == 'prive'" @click="showModal2 = true; showModal2Value = section[0]">+</button> 
              <button v-if="typeSectionProps == 'prive'" @click="supprimerSection(section[0])">X</button>
            </li>
          </ul>
        </b-col>
        <b-col cols="9" id="zoneTchatCSS">
          <chatBox v-if="sectionProps != ''" class="tchatZoneCSS" :sectionActive='sectionProps'/>
          <div id="imgFond" v-if="sectionProps == ''"><img src="../assets/icons/mascot.png" alt=""></div>
          
        </b-col>
      </b-row>
    </b-container>
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
  props: ['typeSectionProps','notificationVu'],

 

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
    ajouterId: '',
    nomSection:''
  }
    
  
    
    

},





methods: {

  afficherSections () {
      console.log("sec aff not",this.notificationVu)
      //privé
      if(this.typeSectionProps == 'prive'){
        var idUser = firebase.auth().currentUser.uid
        var where = 'membres.' + idUser
        db.collection('sections').where('type', '==', this.typeSectionProps).where(where, '==', true).onSnapshot(querySnapshot => {
        this.listeSections = []
        querySnapshot.forEach(doc => {

            //check notif
            var infoMsg

            if(this.notificationVu[0] != undefined){
            for (var i = this.notificationVu.length - 1; i >= 0; i--) {
              if(this.notificationVu[i][0] == doc.id){
                infoMsg = [doc.id,doc.data().nom,true]
                break
              }
              else{
                infoMsg = [doc.id,doc.data().nom,false]
              }
            }
          }
          else{
            infoMsg = [doc.id,doc.data().nom,false]
          }

          this.listeSections.push(infoMsg)

        })
  })

      }
      //perso
      else if (this.typeSectionProps == 'perso') {
         db.collection('users').onSnapshot(querySnapshot => {
        this.listeSections = []
        var idUser = firebase.auth().currentUser.uid
        querySnapshot.forEach(doc => {




          if(doc.id == idUser){} 
           else {
            var infoMsg
            var check = false
            db.collection('sections').where('type', '==', 'perso').where('membres.'+ doc.id ,'==', true).where('membres.'+ idUser ,'==', true).get().then(querySnapshot2 => {
              querySnapshot2.forEach(doc2 => {
                if(doc2.data().vu[idUser] != true){
                  infoMsg = [doc.id,doc.data().nom, true]
                  check = true
                }
                else{
                  infoMsg = [doc.id,doc.data().nom, false]
                  check = true
                }
              
            });
              if(check == false){infoMsg = [doc.id,doc.data().nom, false]}
              this.listeSections.push(infoMsg)
          })

          }
        })

      })
      }


      //global
      else {
      db.collection('sections').where('type', '==', this.typeSectionProps).onSnapshot(querySnapshot => {
        this.listeSections = []
        querySnapshot.forEach(doc => {
           var infoMsg
           

            if(this.notificationVu[0] != undefined){
            for (var i = this.notificationVu.length - 1; i >= 0; i--) {

              if(this.notificationVu[i][0] == doc.id){
                infoMsg = [doc.id,doc.data().nom,true]
                break
              }
              else{
                infoMsg = [doc.id,doc.data().nom,false]
              }
            }
          }
          else{
            infoMsg = [doc.id,doc.data().nom,false]
          }

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

      this.nomMembre = ''



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
        var idUser = firebase.auth().currentUser.uid
          //check si le chat est existe
        var creation = false
              db.collection("sections").where("membres." + id, "==", true).where("membres." + idUser, "==", true).where("type", "==" , "perso")
                .get()
                .then(querySnapshot => {
                    querySnapshot.forEach(doc => {
                        // doc.data() is never undefined for query doc snapshots
                        console.log(doc.id, " => ", doc.data());
                        creation = true
                        this.sectionProps = doc.id
                    });
                      //creation
                        if(creation == false){
                            var membres = {}
                            membres[id] = true
                            membres[idUser] = true
                              db.collection("sections").add({
                          type: this.typeSectionProps,
                          membres: membres
                          })
                          .then(docRef => {
                              this.sectionProps = doc.id
                              console.log("Document written with ID: ", docRef.id);
                              

                          })
                          .catch(error => {
                              console.error("Error adding document: ", error);
                          });

                          


                             }



                })
                .catch(error => {
                    console.log("Error getting documents: ", error);
                });


     


      }
 
      }   
},

created () {
  this.afficherSections()
  this.recuperationUser()
},

watch: {
  typeSectionProps: function ()  {
          this.typeSection = this.typeSectionProps;
          this.afficherSections()

        },
  notificationVu: function () {
       this.afficherSections()
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
  #bodyListeChat{
    height: 90vh;
    margin:0 auto;
    padding: 0;
    background-color: #16222E
  }


 

 .listeSectionsCSS{
    cursor: pointer;
    padding:5px;
    background-color: #16222E;
    border-radius:10px;
    margin-left: -15px;
 }
 .listeSectionsCSS:hover{
    filter: grayscale(70%);
 }
 .selection{
  filter: grayscale(70%);
 }

 .listeChatCSS {
    background-color: #16222E;
    list-style: none;
    margin-top: 10px;

 }
  .listeChatCSS>li{
    cursor: pointer;
    padding:5px;
    background-color: #16222E;
    border-radius:10px;
  }
 .popupCSS {
    position: absolute;
    margin:auto;
    background-color: #e8d6c9;
    z-index: 1000;
    padding: 20px;
 }
 .newCSS2 {
    list-style: disc;
    text-transform: uppercase;
    color:orange;
 }
  #ajouterSectionCSS{
    background-color: #0D181E;
    height: 38px;
    margin-bottom: 15px;
  }

  #ajouterSectionCSS>img{
    height: 25px;
  }

  #titreTypeSection{
    text-transform: uppercase;
    text-align: center;
    padding:10px;
    margin: 10px;
    background-color: #0D181E;
    border-radius: 10px;
  }
  #titreSection{
    position: absolute;
    margin-top: -9vh;
    margin-left:17vw;
    text-align: center;
    z-index: 2;
    color:#157676;
    text-transform: uppercase;
    font-size: 40px;
  }
/*tchat*/
 .tchatZoneCSS{
  width: ;idth: 100%;
 }
 #zoneTchatCSS{
  background-color: #1F303A;
  height: 90vh;
  width: 100%;
  margin:0 auto;
  padding:0}
#imgFond{
  height: 100%;
  width: 100%;
  text-align: center;
  line-height: 90vh

  }
#zoneTchatCSS>div>img{
  margin:auto;


 }
 


 b-modal{
  overflow: hidden;
 }


</style>
