<script setup>
  import { ref } from 'vue';
  import { RouterLink, RouterView, useRouter} from 'vue-router';
  import { storeToRefs } from 'pinia'
  import { useGeneralStore } from '../stores/general.js'
  import { SteadyAPI } from '../utils/api/platform.js'
  import { fileNameToTitle } from '../utils/utils.js'

  import LogoMark from '../components/logos/LogoMark.vue';
  import ThreeDotsIcon from '../components/icons/ThreeDotsIcon.vue';
  import FileIcon from '../components/icons/FileIcon.vue';
  import FileTextIcon from '../components/icons/FileTextIcon.vue';
  import ArrowDownIcon from '../components/icons/ArrowDownIcon.vue';
  import ArrowSquareOutIcon from '../components/icons/ArrowSquareOutIcon.vue';
  import PlusIcon from '../components/icons/PlusIcon.vue';
  import ImagesSquareIcon from '../components/icons/ImagesSquareIcon.vue';
  import TagIcon from '../components/icons/TagIcon.vue';
  import FolderNotchPlusIcon from '../components/icons/FolderNotchPlusIcon.vue';
  import ChatDotsIcon from '../components/icons/ChatDotsIcon.vue';
  import GlobeIcon from '../components/icons/GlobeIcon.vue';

  const generalStore = useGeneralStore();
  const steadyAPI = SteadyAPI();
  const { currentSite } = storeToRefs(generalStore);
  const { changeCurrentSite } = generalStore; 

  const router = useRouter();
  const websites = ref([]);
  const currentWebsite = ref('');
  const showWebsiteDropdown = ref(false);
  const showInfoMenu = ref(false);

  (function() {
    // On load, set view to posts
    router.push({path: '/posts'});
    // And load content
    loadSiteContent();
  })();

  function loadSiteContent() {
    // TODO: make sure the current site is there
    steadyAPI.doesFileExistInPrivate('steady.config.json').then(fileExsits => {
      if (fileExsits) {
          // Get the Current website
          steadyAPI.readFileInPrivate("steady.config.json").then(fileData => {
          currentWebsite.value = fileNameToTitle(JSON.parse(fileData.data).currentWebsite);
          currentSite.value = currentWebsite.value;
          // Get a list of all websites by looping over the dirs and add them to array for the dropdown
          steadyAPI.getPathTo('documents').then(path => {
            steadyAPI.getDirsIn(path + "/SteadyCMS/sites/").then( dirs => {
             // console.log(dirs)
              if (dirs != "error" && dirs.length != 0) {

                for (let i = 0; i < dirs.length; i++) {
                  steadyAPI.doesFileExist("/sites/" + dirs[i] + '/hugo.toml').then(fileExsits => {
                    if (fileExsits && dirs[i] != currentWebsite.value.toLowerCase()) {
                      websites.value.splice(0,0, { "name": fileNameToTitle(dirs[i]), "path": dirs[i], });
                    }
                  });
                }
              } else {
                  // Delete steady.config.json
                  SteadyAPI.deleteFileInPrivate("steady.config.json").then(x => {
                  // They have no websites (have them make one)
                  createNewWebsite(false);
                });
              }
            });
          });
        });
      } else { 
        // They have no websites (have them make one)
        createNewWebsite(false);
      }
    });
  }

  function changeCurrentWebsite(websiteName) { // TODO: when changing site the server must be stop before changed
    console.log(websiteName);
     const obj = {"currentWebsite": websiteName};
     steadyAPI.saveToFileToPrivate(JSON.stringify(obj), "/", "steady.config.json").then(x => {
        websites.value = [];
        router.push({path: '/posts'});
        dropdownState.value = false;
        loadSiteContent();
        changeCurrentSite(websiteName);
    });
  }

  function createNewWebsite(projects) {
    router.push({path: '/new-website', 
      query: {
        hasProjects: projects
      }
    });
  }

  function openSteadyCMSWebsiteURL() {
    steadyAPI.previewInNewBrowserTab('https://steadycms.github.io');
  }

  function openGithubIssuesURL() {
    steadyAPI.previewInNewBrowserTab('https://github.com/SteadyCMS/SteadyCMS/issues/new/choose');
  }

  function openBuyCoffeeURL() {
    steadyAPI.previewInNewBrowserTab('');
  }

</script>

<template>
  <div class="relative flex flex-row"> 
    <aside class="relative h-screen max-w-xs w-2/5 bg-black lg:w-1/4">
      <div class="flex flex-row items-center justify-between border-b border-tint-10 px-6 py-2">
        <div class="flex h-12 w-12 items-center justify-center rounded-full border border-accent-glow/50">
          <LogoMark class="w-5 h-5" />
        </div>
        <button @click="showInfoMenu = !showInfoMenu" class="rounded-lg border p-1.5 ease-in-out duration-300 hover:border-tint-10" :class="[showInfoMenu ? 'border-tint-10' :  'border-transparent']">
          <ThreeDotsIcon class="h-6 w-6 fill-white/80" />
        </button>
        <div v-if="showInfoMenu" class="absolute right-6 top-10 my-4 w-48 list-none rounded-lg border border-tint-10 bg-black text-base z-50">
          <ul class="py-1 w-full">
            <li @click="openSteadyCMSWebsiteURL" class="flex flex-row items-center text-white font-medium text-xs cursor-pointer rounded-md px-3 py-2 ease-in-out duration-300 hover:bg-accent-glow">
              <GlobeIcon class="fill-white w-5 h-5 mr-2" /> Visit website
            </li>
            <li @click="openGithubIssuesURL" class="flex flex-row items-center text-white font-medium text-xs cursor-pointer rounded-md px-3 py-2 ease-in-out duration-300 hover:bg-accent-glow">
              <ChatDotsIcon class="fill-white w-5 h-5 mr-2" /> Submit issue or feature
            </li>
          </ul>
          <div class="mt-1 border-t border-tint-10 pt-3 pb-1 px-3">
            <p class="text-white font-medium text-xs">SteadyCMS v0.1.0-alpha</p>
            <p class="text-white/70 text-[10px]">&copy; 2023, The SteadyCMS team.</p>
          </div>
        </div> 
      </div>
      <div class="px-6">
        <div class="flex flex-row space-x-2 mt-5">
          <div class="relative w-full min-w-0">
            <!-- Website selector -->
            <button @click="showWebsiteDropdown =! showWebsiteDropdown" class="flex w-full flex-grow items-center justify-between rounded-lg border border-tint-10 px-3 py-2 ease-in-out duration-300 hover:bg-accent-glow">
              <div class="flex max-w-xs items-center space-x-3 overflow-hidden">
                <!-- <img class="h-5 w-5 rounded-sm" src="https://picsum.photos/200" alt="" /> -->
                <LogoMark class="w-4 h-4 rounded-sm border border-tint-10" />
                <p class="max-w-xs overflow-hidden text-ellipsis whitespace-nowrap text-sm font-medium text-white">
                  {{ currentWebsite }}
                </p>
              </div>
              <ArrowDownIcon class="fill-white w-3 h-3 ml-1" :class="{'rotate-180 duration-300': showWebsiteDropdown, 'duration-300' : !showWebsiteDropdown}"/>
            </button>
            <div class="absolute left-0 top-6 z-40 my-4 w-full list-none rounded-lg border border-tint-10 bg-black text-base" id="websiteDropdown" :class="{'opacity-100': showWebsiteDropdown, 'visible': showWebsiteDropdown, 'opacity-0': !v, 'hidden': !showWebsiteDropdown}">
              <ul class="space-y-1 py-1 w-full" aria-labelledby="websiteDropdown">
                <li v-for="site in websites" :key="site.path" @click="changeCurrentWebsite(site.path)" class="w-full cursor-pointer rounded-md px-3 py-2 ease-in-out duration-300 hover:bg-accent-glow">
                  <div class="flex max-w-xs items-center space-x-3 overflow-hidden">
                    <!-- <img class="h-5 w-5 rounded-sm" src="https://picsum.photos/200?seed=32" alt="" /> -->
                    <LogoMark class="w-4 h-4 rounded-sm border border-tint-10" />
                    <p class="overflow-hidden text-ellipsis whitespace-nowrap text-sm font-medium text-white">
                      {{ site.name }}
                    </p>
                  </div>
                </li>
                <!-- New website btn -->
                <li @click="createNewWebsite(true)" class="w-full cursor-pointer rounded-md px-3 py-2 ease-in-out duration-300 hover:bg-accent-glow">
                  <button class="flex items-center space-x-3 w-full">
                    <FolderNotchPlusIcon class="h-5 w-5 fill-white"/>
                    <p class="text-sm font-medium text-white">New website</p>
                  </button>
                </li>
              </ul>
            </div>
          </div>
          <!-- Preview website btn -->
          <button @click="" class="relative py-1.5 px-2.5 border border-tint-10 rounded-lg hover:bg-accent-glow ease-in-out duration-300">
            <ArrowSquareOutIcon class="relative w-4 h-4 fill-white"/>
          </button>
        </div>
        <div class="flex flex-col space-y-1 mt-10">
          <RouterLink class="flex items-center rounded-lg px-3 py-2 text-tint-2 fill-tint-3 font-medium hover:text-tint-2/90 ease-in-out duration-300"
            active-class="active-rl"
            to="/posts">
            <FileTextIcon class="w-5 h-5 mr-3" /> Posts
          </RouterLink>
          <!-- <RouterLink class="flex items-center rounded-lg px-3 py-2 text-tint-2 fill-tint-3 font-medium hover:text-tint-2/90 ease-in-out duration-300"
            active-class="active-rl" 
            to="/pages">
            <FileIcon class="w-5 h-5 mr-3" /> Pages
          </RouterLink>
          <RouterLink class="flex items-center rounded-lg px-3 py-2 text-tint-2 fill-tint-3 font-medium hover:text-tint-2/90 ease-in-out duration-300" 
            active-class="active-rl"
            to="/tags">
            <TagIcon class="w-5 h-5 mr-3" /> Tags
          </RouterLink> -->
          <RouterLink class="flex items-center rounded-lg px-3 py-2 text-tint-2 fill-tint-3 font-medium hover:text-tint-2/90 ease-in-out duration-300" 
            active-class="active-rl"
            to="/media">
            <ImagesSquareIcon class="w-5 h-5 mr-3" /> Media
          </RouterLink>
        </div>
      </div>
    </aside>

    <main class="relative flex flex-col h-screen w-full">
      <div class="border-b border-tint-1 py-8">
      </div>
      <div class="relative bg-tint h-full px-10 pt-5">
        <RouterView />
      </div>
    </main>
  </div>
</template>