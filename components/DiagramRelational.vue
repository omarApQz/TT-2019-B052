<template>
  <div>
    <v-navigation-drawer app clipped permanent>
      <v-list nav dense>
        <v-list-item link @click="saveModel()">
          <v-list-item-icon>
            <v-icon>mdi-content-save</v-icon>
          </v-list-item-icon>
          <v-list-item-title>Save</v-list-item-title>
        </v-list-item>

        <v-list-item link @click="loadModel()">
          <v-list-item-icon>
            <v-icon>mdi-file-download</v-icon>
          </v-list-item-icon>
          <v-list-item-title>Load</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-container fluid fill-height>
      <v-row>
        <v-col>
          <div
            style="width: 100%; display: flex; justify-content: space-between"
          >
            <div
              id="myDiagramDiv"
              style="width: 100%; display: flex;border: solid 1px black; "
            ></div>
          </div>
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <v-textarea v-model="diagramaObtenido" auto-grow></v-textarea>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import go from 'gojs'
import 'gojs/extensionsJSM/Figures'
import { mapGetters } from 'vuex'

export default {
  data() {
    return {
      myDiagram: '',
      myPalette: ''
    }
  },
  computed: {
    ...mapGetters({
      thereIsDiagram: 'diagramRelational/existDiagram',
      diagramaObtenido: 'diagramRelational/getDiagram'
    })
  },
  mounted() {
    const $ = go.GraphObject.make // for conciseness in defining templates

    this.myDiagram = $(
      go.Diagram,
      'myDiagramDiv', // must name or refer to the DIV HTML element
      {
        allowDelete: false,
        allowCopy: false,
        layout: $(go.ForceDirectedLayout),
        'undoManager.isEnabled': true
      }
    )

    const colors = {
      red: '#be4b15',
      green: '#52ce60',
      blue: '#6ea5f8',
      lightred: '#fd8852',
      lightblue: '#afd4fe',
      lightgreen: '#b9e986',
      pink: '#faadc1',
      purple: '#d689ff',
      orange: '#fdb400'
    }

    // the template for each attribute in a node's array of item data
    const itemTempl = $(
      go.Panel,
      'Horizontal',
      $(
        go.Shape,
        {
          desiredSize: new go.Size(15, 15),
          strokeJoin: 'round',
          strokeWidth: 3,
          stroke: null,
          margin: 2
        },
        new go.Binding('figure', 'figure'),
        new go.Binding('fill', 'color'),
        new go.Binding('stroke', 'color')
      ),
      $(
        go.TextBlock,
        {
          stroke: '#333333',
          font: 'bold 14px sans-serif'
        },
        new go.Binding('text', 'name')
      )
    )

    // define the Node template, representing an entity
    this.myDiagram.nodeTemplate = $(
      go.Node,
      'Auto', // the whole node panel
      {
        selectionAdorned: true,
        resizable: true,
        layoutConditions: go.Part.LayoutStandard & ~go.Part.LayoutNodeSized,
        fromSpot: go.Spot.AllSides,
        toSpot: go.Spot.AllSides,
        isShadowed: true,
        shadowOffset: new go.Point(3, 3),
        shadowColor: '#C5C1AA'
      },
      new go.Binding('location', 'location').makeTwoWay(),
      // whenever the PanelExpanderButton changes the visible property of the "LIST" panel,
      // clear out any desiredSize set by the ResizingTool.
      new go.Binding('desiredSize', 'visible', function(v) {
        return new go.Size(NaN, NaN)
      }).ofObject('LIST'),
      // define the node's outer shape, which will surround the Table
      $(go.Shape, 'RoundedRectangle', {
        fill: 'white',
        stroke: '#eeeeee',
        strokeWidth: 3
      }),
      $(
        go.Panel,
        'Table',
        { margin: 8, stretch: go.GraphObject.Fill },
        $(go.RowColumnDefinition, {
          row: 0,
          sizing: go.RowColumnDefinition.None
        }),
        // the table header
        $(
          go.TextBlock,
          {
            row: 0,
            alignment: go.Spot.Center,
            margin: new go.Margin(0, 24, 0, 2), // leave room for Button
            font: 'bold 16px sans-serif'
          },
          new go.Binding('text', 'key')
        ),
        // the collapse/expand button
        $(
          'PanelExpanderButton',
          'LIST', // the name of the element whose visibility this button toggles
          { row: 0, alignment: go.Spot.TopRight }
        ),
        // the list of Panels, each showing an attribute
        $(
          go.Panel,
          'Vertical',
          {
            name: 'LIST',
            row: 1,
            padding: 3,
            alignment: go.Spot.TopLeft,
            defaultAlignment: go.Spot.Left,
            stretch: go.GraphObject.Horizontal,
            itemTemplate: itemTempl
          },
          new go.Binding('itemArray', 'items')
        )
      ) // end Table Panel
    ) // end Node

    // define the Link template, representing a relationship
    this.myDiagram.linkTemplate = $(
      go.Link, // the whole link panel
      {
        selectionAdorned: true,
        layerName: 'Foreground',
        reshapable: true,
        routing: go.Link.AvoidsNodes,
        corner: 5,
        curve: go.Link.JumpOver
      },
      $(
        go.Shape, // the link shape
        { stroke: '#303B45', strokeWidth: 2.5 }
      ),
      $(
        go.TextBlock, // the "from" label
        {
          textAlign: 'center',
          font: 'bold 14px sans-serif',
          stroke: '#1967B3',
          segmentIndex: 0,
          segmentOffset: new go.Point(NaN, NaN),
          segmentOrientation: go.Link.OrientUpright
        },
        new go.Binding('text', 'text')
      ),
      $(
        go.TextBlock, // the "to" label
        {
          textAlign: 'center',
          font: 'bold 14px sans-serif',
          stroke: '#1967B3',
          segmentIndex: -1,
          segmentOffset: new go.Point(NaN, NaN),
          segmentOrientation: go.Link.OrientUpright
        },
        new go.Binding('text', 'toText')
      )
    )

    // create the model for the E-R diagram
    const nodeDataArray = [
      {
        key: 'Products',
        items: [
          {
            name: 'ProductID',
            iskey: true,
            figure: 'Decision',
            color: colors.red
          },
          {
            name: 'ProductName',
            iskey: false,
            figure: 'Hexagon',
            color: colors.blue
          },
          {
            name: 'SupplierID',
            iskey: false,
            figure: 'Decision',
            color: 'purple'
          },
          {
            name: 'CategoryID',
            iskey: false,
            figure: 'Decision',
            color: 'purple'
          }
        ]
      },
      {
        key: 'Suppliers',
        items: [
          {
            name: 'SupplierID',
            iskey: true,
            figure: 'Decision',
            color: colors.red
          },
          {
            name: 'CompanyName',
            iskey: false,
            figure: 'Hexagon',
            color: colors.blue
          },
          {
            name: 'ContactName',
            iskey: false,
            figure: 'Hexagon',
            color: colors.blue
          },
          {
            name: 'Address',
            iskey: false,
            figure: 'Hexagon',
            color: colors.blue
          }
        ]
      },
      {
        key: 'Categories',
        items: [
          {
            name: 'CategoryID',
            iskey: true,
            figure: 'Decision',
            color: colors.red
          },
          {
            name: 'CategoryName',
            iskey: false,
            figure: 'Hexagon',
            color: colors.blue
          },
          {
            name: 'Description',
            iskey: false,
            figure: 'Hexagon',
            color: colors.blue
          },
          {
            name: 'Picture',
            iskey: false,
            figure: 'TriangleUp',
            color: colors.pink
          }
        ]
      },
      {
        key: 'Order Details',
        items: [
          {
            name: 'OrderID',
            iskey: true,
            figure: 'Decision',
            color: colors.red
          },
          {
            name: 'ProductID',
            iskey: true,
            figure: 'Decision',
            color: colors.red
          },
          {
            name: 'UnitPrice',
            iskey: false,
            figure: 'Circle',
            color: colors.green
          },
          {
            name: 'Quantity',
            iskey: false,
            figure: 'Circle',
            color: colors.green
          },
          {
            name: 'Discount',
            iskey: false,
            figure: 'Circle',
            color: colors.green
          }
        ]
      }
    ]
    const linkDataArray = [
      { from: 'Products', to: 'Suppliers', text: '0..N', toText: '1' },
      { from: 'Products', to: 'Categories', text: '0..N', toText: '1' },
      { from: 'Order Details', to: 'Products', text: '0..N', toText: '1' }
    ]
    this.myDiagram.model = $(go.GraphLinksModel, {
      copiesArrays: true,
      copiesArrayObjects: true,
      nodeDataArray,
      linkDataArray
    })

    /* if (this.thereIsDiagram) {
      this.myDiagram.model = new go.GraphLinksModel()
      this.myDiagram.model = go.Model.fromJson(this.diagramaObtenido)
      const pos = this.myDiagram.model.modelData.position
      if (pos) this.myDiagram.initialPosition = go.Point.parse(pos)
    } else {
      this.myDiagram.model = new go.GraphLinksModel()
    } */
  },
  beforeDestroy() {
    // this.saveModel()
  },
  middleware: 'authenticated',
  layout: 'workspace', // layout de la aplicación (esto es de nuxt)
  methods: {
    saveDiagramProperties() {
      this.myDiagram.model.modelData.position = go.Point.stringify(
        this.myDiagram.position
      )
    },
    saveModel() {
      this.saveDiagramProperties()
      this.savedModel = this.myDiagram.model.toJson()
      this.myDiagram.isModified = false
      //  console.log('savedModel: ' + this.savedModel)
      this.$store.dispatch('diagramRelational/save', {
        savedModel: this.savedModel
      })
    },
    loadModel() {}
  }
}
</script>

<style>
#myDiagramDiv {
  height: 1000px;
}
</style>
