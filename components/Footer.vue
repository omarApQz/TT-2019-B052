<template>
  <!--<v-footer app color="primary lighten-1" padless>
    <v-row justify="center" no-gutters>
      <v-col class="primary darken-1 py-4 text-center white--text" cols="12">
        {{ new Date().getFullYear() }} —
        <strong>Instituto Politécnico Nacional</strong>
      </v-col>
    </v-row>
  </v-footer>-->
  <v-footer app color="primary darken-1 py-4 text-center white--text">
    <v-spacer />
    <span>Instituto Politécnico Nacional - {{ new Date().getFullYear() }}</span>
    <v-spacer />
  </v-footer>
</template>

<script>
export default {}
</script>
