<script>
export default {
    data: () => ({
        dependsOnValue: null,
        watcherDebounce: null,
        watcherDebounceTimeout: 200,
    }),

    created() {
        if (this.field.dependsOn !== undefined) {
            this.registerDependencyWatchers(this.$root);
        }
    },

    beforeDestroy() {
        if (this.watcherDebounce) {
            clearTimeout(this.watcherDebounce);
        }
    },

    methods: {
        registerDependencyWatchers(root) {
            root.$children.forEach(component => {
                if (this.componentIsDependency(component)) {
                    if (component.selectedResourceId !== undefined) {
                        // BelongsTo field
                        console.log('BelongsTo');
                        console.log(component.selectedResourceId);
                        component.$watch('selectedResourceId', this.dependencyWatcher, {immediate: true});
                        this.dependencyWatcher(component.selectedResourceId);
                    }
                    else
                    if (component.selectedResource !== undefined) {
                        // BelongsTo field
                        console.log('Searchable_Belongs_To');
                        console.log(component.selectedResource);
                        console.log(component.selectedResource.id);
                        component.$watch('selectedResource', this.dependencyWatcher, {immediate: true});
                        this.dependencyWatcher(component.selectedResource.id);
                    } else
                    if (component.value !== undefined) {
                        // Text based fields
                        console.log('Value');
                        console.log(component.value);
                        component.$watch('value', this.dependencyWatcher, {immediate: true});
                        this.dependencyWatcher(component.value);
                    }
                }

                this.registerDependencyWatchers(component);
            })
        },
        componentIsDependency(component) {
            if (component.field === undefined) {
                return false;
            }

            return component.field.attribute === this.field.dependsOn;
        },
        dependencyWatcher(value) {
            clearTimeout(this.watcherDebounce);
            this.watcherDebounce = setTimeout(() => {
                if (value === this.dependsOnValue) {
                    return;
                }

                this.dependsOnValue = value;

                this.clearSelection();
                this.$nextTick(() => {
                    this.initializeComponent();
                });

                this.watcherDebounce = null;
            }, this.watcherDebounceTimeout);
        },
    },

    computed: {
        queryParams() {
            return {
                params: {
                    current: this.selectedResourceId,
                    first: this.initializingWithExistingResource,
                    search: this.search,
                    withTrashed: this.withTrashed,
                    dependsOnValue: this.dependsOnValue,
                },
            };
        },
    },
}
</script>
