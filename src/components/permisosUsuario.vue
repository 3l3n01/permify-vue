<template>
    <v-simple-table>
        <template v-slot:default>
            <thead>
                <tr>
                    <th class="text-left">
                        Usuario
                    </th>
                    <th class="text-left">
                        Galeria
                    </th>
                    <th class="text-left">
                        Mensajes
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="item in desserts" :key="item.name">
                    <td>{{ item.name }}</td>
                    <td>
                        <v-switch v-model="item.galeria" color="success" hide-details :loading="item.loading"
                            @click.prevent="permission(item, 'Galeria')"></v-switch>
                    </td>
                    <td>
                        <v-switch v-model="item.mensajes" color="success" hide-details :loading="item.loading"
                            @click.prevent="permission(item, 'Mensajes')"></v-switch>
                    </td>
                </tr>
            </tbody>
        </template>
    </v-simple-table>
</template>

<script>
export default {
    data() {
        return {
            desserts: [],
            tenantId: 't1',
        }
    },
    mounted() {
        var db = localStorage.getItem('myDB');
        if (db) {
            this.desserts = JSON.parse(db);
        } else {
            this.desserts = [
                {
                    name: 'YO',
                    id: 159,
                    galeria: false,
                    mensajes: false,
                    loading: false
                },
                {
                    name: 'Ice cream sandwich',
                    id: 237,
                    galeria: false,
                    mensajes: false,
                    loading: false
                },
                {
                    name: 'Eclair',
                    id: 262,
                    galeria: false,
                    mensajes: false,
                    loading: false
                },
                {
                    name: 'Cupcake',
                    id: 305,
                    galeria: false,
                    mensajes: false,
                    loading: false
                },
                {
                    name: 'Gingerbread',
                    id: 356,
                    galeria: false,
                    mensajes: false,
                    loading: false
                },
                {
                    name: 'Jelly bean',
                    id: 375,
                    galeria: false,
                    mensajes: false,
                    loading: false
                },
                {
                    name: 'Lollipop',
                    id: 392,
                    galeria: false,
                    mensajes: false,
                    loading: false
                },
                {
                    name: 'Honeycomb',
                    id: 408,
                    galeria: false,
                    mensajes: false,
                    loading: false
                },
                {
                    name: 'Donut',
                    id: 452,
                    galeria: false,
                    mensajes: false,
                    loading: false
                },
                {
                    name: 'KitKat',
                    id: 518,
                    galeria: false,
                    mensajes: false,
                    loading: false
                },
            ];
            localStorage.setItem('myDB', JSON.stringify(this.desserts));
        }
    },
    methods: {
        save: function () {
            // Guardar lo que tenga desserts
            localStorage.removeItem('myDB');
            localStorage.setItem('myDB', JSON.stringify(this.desserts));
        },
        async permission(item, screen) {
            item.loading = true;
            console.log("SCREEN : ", screen, ' - ', item[screen.toLowerCase()])
            // Ver si es agregar o quitar los permisos
            if (item[screen.toLowerCase()]) {
                const rawResponse = await fetch('http://localhost:3476/v1/tenants/t1/relationships/write', {
                    method: 'POST',
                    headers: {
                        'Accept': 'application/json',
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        "metadata": {
                            "schema_version": ""
                        },
                        "tuples": [
                            {
                                "entity": {
                                    "type": "screens",
                                    "id": screen.toLowerCase()
                                },
                                "relation": "colaborador",
                                "subject": {
                                    "type": "user",
                                    "id": item.id + "",
                                    "relation": ""
                                }
                            }
                        ]
                    })
                });
                const content = await rawResponse.json();
                console.log(content)
                setTimeout(() => {
                    item.loading = false;
                    this.save();
                }, 500);
            } else {
                const rawResponse = await fetch('http://localhost:3476/v1/tenants/t1/relationships/delete', {
                    method: 'POST',
                    headers: {
                        'Accept': 'application/json',
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        "filter": {
                            "entity": {
                                "type": "screens",
                                "ids": [screen.toLowerCase()]
                            },
                            "relation": "colaborador",
                            "subject": {
                                "type": "user",
                                "ids": [item.id+""],
                                "relation": ""
                            }
                        }
                    })
                });
                const content = await rawResponse.json();
                this.save();
                console.log(content)
                setTimeout(() => {
                    item.loading = false;
                    this.save();
                }, 500);
            }
        }
    }
}
</script>
