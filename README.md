# Vue Data table component 

## Usage

```
Download the repository and add the components wherever you want in your project

The next step is to use the component, example:

```javascript

<template>
	<data-table :pagination="pagination"
				:data="data"
				:columns="columns"
				:settings="settings"
				@per-page="perPage"
				@throw-page="changePage"
				@sort="changeSort"
				@filter="changeFilter"
	></data-table>
</template>

<script>
	import Datatable from './DataTable.vue'
	
	new Vue({
		components: {
			'data-table': Datatable
		}
	})
</script>

```
	For more information please visit [documentation in the Wiki pages](https://github.com/przemyslawDev/vdtc/wiki).
```

