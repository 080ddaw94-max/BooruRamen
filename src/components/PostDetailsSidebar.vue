<template>
  <div 
    class="absolute top-0 left-0 w-80 h-full bg-transparent backdrop-blur-sm border-r border-gray-700 overflow-y-auto z-50 transition-transform duration-300 ease-in-out"
    :style="{ transform: show ? 'translateX(0)' : 'translateX(-100%)' }"
  >
    <div class="p-4" style="padding-top: calc(1rem + env(safe-area-inset-top, 0)); padding-bottom: calc(5rem + env(safe-area-inset-bottom, 0));">
      <h2 class="text-xl font-bold mb-4">Post Details</h2>
      
      <div class="space-y-4" v-if="post">
        <div>
          <h3 class="text-sm font-medium text-gray-400">ID</h3>
          <p>{{ post.id }}</p>
        </div>
        
        <div>
          <h3 class="text-sm font-medium text-gray-400">Uploader</h3>
          <p>{{ post.uploader_name || 'Unknown' }}</p>
        </div>
        
        <div>
          <h3 class="text-sm font-medium text-gray-400">Rating</h3>
          <p class="capitalize">{{ getRatingFromCode(post.rating) }}</p>
        </div>
        
        <div>
          <h3 class="text-sm font-medium text-gray-400">Score</h3>
          <p>{{ post.score || 0 }}</p>
        </div>
        
        <div>
          <h3 class="text-sm font-medium text-gray-400">Media Info</h3>
          <p>{{ (post.file_ext || 'Unknown').toUpperCase() }} - {{ formatFileSize(post.file_size) }}</p>
          <p>{{ post.image_width || '?' }}×{{ post.image_height || '?' }}</p>
        </div>
        
        <!-- Only render tag_string if we don't have categories OR if debug/general view is needed. -->
        <!-- Since we are separating them, let's use the categories. -->
        <TagSection v-if="post.tag_string_artist" title="Artist Tags" :tagString="post.tag_string_artist" colorClass="bg-pink-900" />
        <TagSection v-if="post.tag_string_character" title="Character Tags" :tagString="post.tag_string_character" colorClass="bg-green-900" />
        <TagSection v-if="post.tag_string_copyright" title="Copyright Tags" :tagString="post.tag_string_copyright" colorClass="bg-blue-900" />
        <TagSection v-if="post.tag_string_meta" title="Meta Tags" :tagString="post.tag_string_meta" colorClass="bg-purple-900" />
        <TagSection v-if="post.tag_string_general" title="General Tags" :tagString="post.tag_string_general" colorClass="bg-gray-700" />
        
        <!-- Fallback if specific tags don't exist but master string does -->
        <TagSection 
          v-if="!post.tag_string_artist && !post.tag_string_character && !post.tag_string_general && post.tag_string" 
          title="All Tags" 
          :tagString="post.tag_string" 
          colorClass="bg-gray-700" 
        />
        
        <div>
          <h3 class="text-sm font-medium text-gray-400">Created</h3>
          <p>{{ post.created_at ? new Date(post.created_at).toLocaleString() : 'Unknown date' }}</p>
        </div>
        
        <div>
          <h3 class="text-sm font-medium text-gray-400">Source</h3>
          <p class="text-pink-400">{{ getSourceName(post.source) }}</p>
        </div>
        
        <div class="flex justify-between items-center mt-4">
          <a  
            :href="post.post_url || (post.id ? `https://danbooru.donmai.us/posts/${post.id}` : '#')" 
            target="_blank" 
            class="text-gray-400 hover:text-white transition-colors p-2 rounded-full hover:bg-gray-800"
            title="Open in browser"
          >
            View in Browser
          </a>
          <button 
            @click="copyLink"
            class="relative text-gray-400 hover:text-white transition-colors p-2 rounded-full hover:bg-gray-800"
            title="Copy link"
          >
            {{ linkCopied ? 'Copied!' : 'Copy Link' }}
            <span 
              v-if="linkCopied" 
              class="absolute top-0 right-0 bottom-0 left-0 bg-green-600 rounded-full flex items-center justify-center text-white"
              style="animation: fadeOut 1.5s forwards;"
            >
              Copied!
            </span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import TagSection from './TagSection.vue';

export default {
  name: 'PostDetailsSidebar',
  components: { TagSection },
  props: {
    show: Boolean,
    post: Object,
  },
  data() {
    return {
      linkCopied: false,
    };
  },
  methods: {
    getRatingFromCode(code) {
      if (!code) return 'Unknown';
      const map = { 'g': 'General', 's': 'Sensitive', 'q': 'Questionable', 'e': 'Explicit' };
      return map[code] || code;
    },
    formatFileSize(bytes) {
      if (!bytes) return 'Unknown size';
      const sizes = ['Bytes', 'KB', 'MB', 'GB'];
      const i = Math.floor(Math.log(bytes) / Math.log(1024));
      return parseFloat((bytes / Math.pow(1024, i)).toFixed(2)) + ' ' + sizes[i];
    },
    getSourceName(sourceStr) {
      if (!sourceStr) return 'Unknown';
      return sourceStr.charAt(0).toUpperCase() + sourceStr.slice(1);
    },
    copyLink() {
      if (!this.post) return;
      const url = this.post.post_url || `https://danbooru.donmai.us/posts/${this.post.id}`;
      navigator.clipboard.writeText(url).then(() => {
        this.linkCopied = true;
        setTimeout(() => {
          this.linkCopied = false;
        }, 1500);
      });
    }
  }
}
</script>
