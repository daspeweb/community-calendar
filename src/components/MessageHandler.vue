
<script>
export default {
  name: "MessageHandler",
  methods: {
    register: (name, callback) => {
      window.addEventListener('message', function (e){
        // console.log(e.data)
        if (e.data.eventName !== name){return}
        callback(e.data, e)
      })
    }
  }
}
</script>
