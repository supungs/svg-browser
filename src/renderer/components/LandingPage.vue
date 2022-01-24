<template>
  <div id="wrapper">
    <nav class="navbar" role="navigation" aria-label="main navigation">
      <div class="navbar-brand">
        <a class="navbar-item" href="http://firolab.com">
          <img id="logo" src="~@/assets/logo.png" alt="electron-vue">
        </a>
      </div>
      <div class="navbar-menu">
        <div class="navbar-start">
          <div class="navbar-item">
            <a @click="goBack" class="button is-primary">Back</a>
          </div>
          <div class="navbar-item breadcrumb" aria-label="breadcrumbs">
            <ul>
              <li v-for="(fldr, i) in breadcrumbs" :key="i" :class="{'is-active': i==(breadcrumbs.length-1)}">
                <a @click="goTo(fldr.path)" href="#" >{{fldr.crumb}}</a>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </nav>
    <!-- <div class="panel path">
      <div class="panel-block">
        <nav class="panel breadcrumb" aria-label="breadcrumbs">
          <ul>
            <li v-for="(fldr, i) in breadcrumbs" :key="i" :class="{'is-active': i==(breadcrumbs.length-1)}">
              <a href="#" >{{fldr}}</a>
            </li>
          </ul>
        </nav>
      </div>
    </div> -->
    <div class="main">
      <div class="columns is-multiline">
        <div v-for="(f, i) in files" @dblclick="onDblClick(f.name)" :key="i" class="column is-2">
          <div class="card">
            <div class="card-image">
              <img v-if="f.type == 'dir'" src="~@/assets/folder.svg" alt="Placeholder image">
              <img v-else-if="f.type == 'disk'" src="~@/assets/disk.svg" alt="Placeholder image">
              <img v-else-if="f.type == 'img'" :src="icon(f)" alt="Placeholder image">
              <img v-else src="~@/assets/file.svg" alt="Placeholder image">
            </div>
            <div class="card-content">
              <div class="content">{{f.name}}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import SystemInformation from './LandingPage/SystemInformation';
  const fs = require('fs');

  export default {
    name: 'landing-page',
    components: { SystemInformation },
    data () {
      return {
        files: [],
        path: 'Computer',
        drives: []
      }
    },
    computed: {
      breadcrumbs (){
        var parts = this.path.split('/');
        var crumbs = [];
        var path = '';
        for(var i = 0; i < parts.length; i++){
          path += (i != 0 ? '/':'') + parts[i];
          crumbs.push({crumb: parts[i], path: path});
        }
        return crumbs;
      }
    },
    methods: {
      onDblClick (name) {
        this.openDir(name);
      },
      openDir (name){
        if (this.path.charAt(this.path.length-1) == '/') this.path += name;
        else if (this.path == 'Computer') this.path =  name + ':/';
        else  this.path += '/' + name;
        this.refresh();
      },
      goBack (){
        this.path = this.path.substring( 0, this.path.lastIndexOf( "/" ));
        if (this.path.lastIndexOf( "/" ) == -1) this.path = 'Computer';
        this.refresh();
      },
      goTo (path){
        this.path = path;
        this.refresh();
      },
      refresh () {
        this.files = [];
        if (this.path == 'Computer'){
          this.files = this.drives;
        } else {
          fs.readdir(this.path, (err, list) => {
            list.forEach(file => {
              var fobj = { name: file, type: this.fileType(file) };
              this.files.push(fobj);
            });
          });
        }
      },
      fileType (f){
        f = f.toLowerCase();
        if (fs.statSync(this.path + '/' + f).isDirectory()) return 'dir';
        else if (f.endsWith('.png') || f.endsWith('.jpg')) return 'img';
        else if (f.endsWith('.svg')) return 'img';
        else return 'other';
      },
      loadDrives (){
        var letters = ['C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K'];
        letters.forEach(ltr => {
          if (fs.existsSync(ltr + ':/')) {
            var fobj = { name: ltr, type: 'disk' };
            this.drives.push(fobj);
          }
        });
      },
      icon (f){
        return this.path + '/' + f.name;
      }
    },
    mounted () {
      this.loadDrives();
      this.refresh();
    }
  }
</script>

<style lang="scss" scoped>
  .panel.path{
    margin-bottom: 1px !important;
  }
  .main{
    padding: 10px;
    min-height: calc(100vh - 123px);
    border: 1px solid #dbdbdb;

    .card{
      box-shadow: 0 0 0;
      .card-image{
        padding: 5px;
      }
      .card-content{
        padding: 5px;
        text-align: center;
      }
    }
    .card:hover{
      background: rgb(163, 192, 255);
    }
  }
</style>
