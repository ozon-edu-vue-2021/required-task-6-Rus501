<script lang="jsx">
import FilterDropdown from './filter-dropdown';
import TablePaginator from './table-paginator';
import DotsLoaderIcon from './dots-loader.svg';

export default {
  name: 'Table',
  props: {
    rows: {
      type: Array,
      default: () => []
    },
    totalPages: {
      type: Number,
      default: 0
    },
    currentPage: {
      type: Number,
      default: 0
    },
    staticPaging: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      sortProp: '',
      sortDirection: '',
      filterProp: '',
      filterText: ''
    }
  },
  computed: {
    sortedRows() {
      let res      

      res = [...this.rows].sort((a, b) => {
        if (!this.sortProp) {
          res = this.rows
          return
        }
        
        if (typeof a[this.sortProp] === 'string') {
          if (a[this.sortProp] < b[this.sortProp]) return this.sortDirection === 'asc' ? -1 : 1
          if (a[this.sortProp] > b[this.sortProp]) return this.sortDirection === 'asc' ? 1 : -1
          return 0
        }

        if (typeof a[this.sortProp] === 'number') {
          return this.sortDirection === 'asc'
            ? a[this.sortProp] - b[this.sortProp]
            : b[this.sortProp] - a[this.sortProp]
        }
      })

      if (this.filterText) {
        res = res.filter(row => String(row[this.filterProp]).search(this.filterText) > -1)
      }

      return res
    }
  },
  methods: {
    toggleSort(prop) {
      this.sortProp = prop
      this.sortDirection = (this.sortDirection === 'desc' || !this.sortDirection) ? 'asc' : 'desc'
    },
    openFilterTooltip(prop = '') {
      this.filterProp = prop
      this.filterText = ''
    },
    setFilterText(e) {
      this.filterText = e.target.value
    },
    renderHead(h, columnsOptions) {
      const { $style, sortProp, sortDirection, filterProp, filterText } = this

      return columnsOptions.map((column) => {
        let sortIcon = 'sort'

        if (sortProp === column.prop) 
          sortIcon = sortDirection === 'asc' ? 'sort-amount-down' : 'sort-amount-up'

        return (
          <th key={column.prop} class={this.$style.headerCell}>
            <div class={$style.headerCellContent}>
              <span>{column.title}</span>
              <font-awesome-icon
                class={$style.sortIcon}
                icon={sortIcon}
                on={{ click: () => this.toggleSort(column.prop) }}
              />
              <FilterDropdown 
                columnProp={column.prop}
                shown={column.prop === filterProp}
                filterText={filterText}

                on={{
                  openFilterTooltip: () => this.openFilterTooltip(column.prop),
                  closeFilterTooltip: () => this.openFilterTooltip(),
                  setFilterText: this.setFilterText,
                }}
              />
            </div>
          </th>
        )
      })
    },
    renderRows(h, columnsOptions) {
      return this.sortedRows.map((row, i) => {
        return (
          <tr key={row.id || i}>{...this.renderColumns(h, row, columnsOptions)}</tr>
        )
      })
    },
    renderColumns(h, row, columnsOptions) {
      return columnsOptions.map(column => {
        return (
          <td key={column.prop} class={this.$style.cell}>
            {row[column.prop]}
          </td>
        )
      })
    },
    getColumnOptions() {
      return this.$slots.default.
        map(column => ({...column.componentOptions.propsData}))
    },
    renderInfPager() {
      const directives = [
        {
          name: 'detect-viewport',
          value: {
            callback: this.$listeners.getPage
          }
        }
      ];

      const style = {
        background: `url("${DotsLoaderIcon}") no-repeat center`
      };

      return (
        <div {...{ class: this.$style.infPager, style, directives }} />
      );
    }
  },
  render(h) {
    const { $style, totalPages, currentPage, staticPaging, $listeners } = this
    const { getPage } = $listeners
    const columnsOptions = this.getColumnOptions()
    const columnsHead = this.renderHead(h, columnsOptions)
    const rows = this.renderRows(h, columnsOptions)

    return (
      <div>
        <table class={$style.table}>
          <thead>{...columnsHead}</thead>
          <tbody>{...rows}</tbody>
        </table>

        {staticPaging
          ? <TablePaginator totalPages={totalPages} currentPage={currentPage} on={{ getPage: getPage }} />
          : this.renderInfPager()
        }

      </div>
    )
  }
}
</script>

<style module>
.table {
  border-spacing: 0;
  margin: 8px;
  /* width: 100%; */
}

.cell {
  text-align: left;
  border-bottom: 1px solid #c8cacc;
  padding: 1rem 1rem;
}

.headerCell {
  composes: cell;
  background: #c7cbcb;
}

.headerCellContent {
  display: flex;
  align-items: center;
}

.sortIcon {
  margin-left: 8px;
  margin-right: 24px;
}

.sortIcon:hover {
  cursor: pointer;
}

.infPager {
  width: 100%;
  height: 32px;
}
</style>