<style>
  form{
    display: block;
    height: 400px;
    width: 400px;
    background: #ccc;
    margin: auto;
    margin-top: 40px;
    text-align: center;
    line-height: 400px;
    border-radius: 4px;
  }

  div.file-listing{
    width: 400px;
    margin: auto;
    padding: 10px;
    border-bottom: 1px solid #ddd;
  }

  div.file-listing img{
    height: 100px;
  }

  div.remove-container{
    text-align: center;
  }

  div.remove-container a{
    color: red;
    cursor: pointer;
  }

  a.submit-button{
    display: block;
    margin: auto;
    text-align: center;
    width: 200px;
    padding: 10px;
    text-transform: uppercase;
    background-color: #CCC;
    color: white;
    font-weight: bold;
    margin-top: 20px;
  }

  progress{
    width: 400px;
    margin: auto;
    display: block;
    margin-top: 20px;
    margin-bottom: 20px;
  }
  .drop-files{
    background-color: aqua;
  }
</style>

<template>
  <div id="file-drag-drop">
    <form ref="fileform">
      <span class="drop-files">Drop the files here!</span>
    </form>

    <progress max="100" :value.prop="uploadPercentage"></progress>

    <v-flex v-for="(file, key) in files" :key="file" class="file-listing">
      <img class="preview" v-bind:ref="'preview'+parseInt( key )"/>
     
      <div class="remove-container">
        <a class="remove" v-on:click="removeFile( key )">Remove</a>
      </div>
    </v-flex>

    <a class="submit-button" v-on:click="submitFiles()" v-show="files.length > 0">Submit</a>
  </div>
</template>

<script>
  export default {
    data(){
      return {
        dragAndDropCapable: false,
        files: [],
        uploadPercentage: 0
      }
    },

    mounted(){

      this.dragAndDropCapable = this.determineDragAndDropCapable();
      if( this.dragAndDropCapable ){
        ['drag', 'dragstart', 'dragend', 'dragover', 'dragenter', 'dragleave', 'drop'].forEach( function( evt ) {
          this.$refs.fileform.addEventListener(evt, function(e){
            e.preventDefault();
            e.stopPropagation();
          }.bind(this), false);
        }.bind(this));
        this.$refs.fileform.addEventListener('drop', function(e){
          for( let i = 0; i < e.dataTransfer.files.length; i++ ){
            this.files.push( e.dataTransfer.files[i] );
            this.getImagePreviews();
          }
        }.bind(this));
      }
    },

    methods: {
      determineDragAndDropCapable(){
        var div = document.createElement('div');
        return ( ( 'draggable' in div )
                || ( 'ondragstart' in div && 'ondrop' in div ) )
                && 'FormData' in window
                && 'FileReader' in window;
      },
      getImagePreviews(){
        for( let i = 0; i < this.files.length; i++ ){
          if ( /\.(jpe?g|png|gif)$/i.test( this.files[i].name ) ) {
            let reader = new FileReader();
            reader.addEventListener("load", function(){
              this.$refs['preview'+parseInt( i )][0].src = reader.result;
            }.bind(this), false);
            reader.readAsDataURL( this.files[i] );
          }else{
            this.$nextTick(function(){
              this.$refs['preview'+parseInt( i )][0].src = '/images/file.png';
            });
          }
        }
      },

      // /*
      //   Submits the files to the server
      // */
      // submitFiles(){
      //   /*
      //     Initialize the form data
      //   */
      //   let formData = new FormData();

      //   /*
      //     Iteate over any file sent over appending the files
      //     to the form data.
      //   */
      //   for( var i = 0; i < this.files.length; i++ ){
      //     let file = this.files[i];

      //     formData.append('files[' + i + ']', file);
      //   }

      //   /*
      //     Make the request to the POST /file-drag-drop URL
      //   */
      //   this.$http.post( 'upload/file-drag-drop',
      //     formData,
      //     {
      //       headers: {
      //           'Content-Type': 'multipart/form-data'
      //       },
      //       onUploadProgress: function( progressEvent ) {
      //         this.uploadPercentage = parseInt( Math.round( ( progressEvent.loaded * 100 ) / progressEvent.total ) );
      //       }.bind(this)
      //     }
      //   ).then(function(){
      //     console.log('SUCCESS!!');
      //   })
      //   .catch(function(){
      //     console.log('FAILURE!!');
      //   });
      // },

      // /*
      //   Removes a select file the user has uploaded
      // */
      removeFile( key ){
        this.files.splice( key, 1 );
      }
    }
  }
</script>