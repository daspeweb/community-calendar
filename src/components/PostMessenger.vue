
<script>
export default {
  name: "PostMessenger",
  methods: {
    postParent: (eventName, data, url) => {
      console.info(`posting to parent ${eventName}`)
      parent.postMessage({...{eventName: eventName}, ...data}, url);
    },
    post: (eventName, data, url) => {
      console.info(`posting ${eventName}`)
      console.log({...{eventName: eventName}, ...data})
      window.postMessage({...{eventName: eventName}, ...data}, url);
    }
  }
}
</script>
