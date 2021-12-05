<script>
export default {
  name: 'table-column',
  props: {
    prop: {
      type: String,
      required: true
    },
    title: {
      type: String,
      default: ''
    }
  }
}
</script>
