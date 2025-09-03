# SuperContainer

An advanced multi-layout container component for Budibase applications that provides flexible layout options including container, grid, tabs, split-view, and field group layouts with comprehensive styling and interaction capabilities.

## 🚀 Features

### Layout Options

- **Container Mode**: Flexbox-based flexible container with direction, alignment, and wrapping
- **Grid Mode**: CSS Grid with configurable columns and rows
- **Tabs Mode**: Tabbed interface with customizable themes and navigation
- **SplitView Mode**: Resizable panels with adjustable ratios
- **Field Group Mode**: Form-based layout with label positioning

### Layout Control

- **Flex Properties**: Grow/shrink modes with configurable flex factors
- **Directional Control**: Row/column layouts with item alignment
- **Spacing Control**: Gap settings for proper spacing
- **Alignment Options**: Horizontal and vertical positioning controls
- **Wrapping Support**: Multi-line arrangements when needed

### Advanced Functionality

- **Collapsible Panels**: Expandable/collapsible containers with custom titles
- **Hover Effects**: Interactive styling on mouse over
- **Event Integration**: Click and right-click event handling
- **Nested Support**: Hierarchical container structures
- **Responsive Design**: Adaptive layouts for different screen sizes

## 🎯 Usage Instructions

### Basic Setup

1. Add the SuperContainer component to your screen
2. Select your layout mode (Container, Grid, Tabs, SplitView, or Field Group)
3. Configure layout-specific settings
4. Add child components and configure their properties
5. Customize styling and interaction options

### Layout Modes

#### Container Mode

- **Direction**: Row (horizontal) or Column (vertical)
- **Alignment**: Stretch, center, flex-start, flex-end, space-between, space-around
- **Gap**: Control spacing between child items
- **Wrapping**: Enable/disable flex-wrap for multi-line layouts
- **Flexible Sizing**: Grow/shrink child items based on content and space

#### Grid Mode

- **Column Count**: Configurable grid columns
- **Row Count**: Configurable grid rows
- **Item Positioning**: Column and row spans for individual components
- **Responsive Grids**: Adaptive column/row adjustments

#### Tabs Mode

- **Theme Selection**: Budibase, Buttons, Negative Buttons, List themes
- **Tab Sizing**: Small, Medium, Large options
- **Icon Support**: Icons for each tab
- **Navigation**: Active tab control and change events
- **Quiet Mode**: Subtle tab styling option

#### SplitView Mode

- **Resizable Panels**: Drag to adjust panel sizes
- **Ratio Control**: Define initial split ratios
- **Multiple Panels**: Support for complex panel arrangements
- **Smooth Interactions**: Fluid resizing animations

### Collapsible Features

#### Collapsible Panels

- **Collapse Direction**: Left, Top, Right, Bottom positioning
- **Custom Titles**: Expanded and collapsed state titles
- **Icon Display**: Custom icons for collapsed state
- **Size Control**: Adjustable collapsed dimensions
- **Smooth Transitions**: Animated expand/collapse

#### Configuration Options

- **Initial State**: Start expanded or collapsed
- **Panel Content**: Full content in expanded view
- **Minimal Display**: Icon and title in collapsed view
- **Interactive Controls**: Expand/collapse by clicking

### Field Group Mode

#### Form Layout

- **Label Positioning**: Left-aligned or above field labels
- **Grid Columns**: Multi-column form layouts
- **Label Width**: Consistent label width for left-aligned layouts
- **Disabled State**: Enable/disable entire field groups

#### Field Management

- **Column Spanning**: Allow fields to span multiple columns
- **Row Management**: Control field positioning
- **Visual Grouping**: Logical field organization

## 🔧 Configuration Properties

### Layout Mode Selection

- **Mode**: Container, Grid, Tabs, SplitView, or Field Group
- **Child Mode**: Container Item, SplitView Item, Tabs Item, Grid Item, or Field Group Item
- **Flex Mode**: Shrink or grow container sizing
- **Collapsible**: Enable/disable collapsible behavior

### Container Layout Settings

- **Direction**: Row or column layout flow
- **Horizontal Align**: Flex-start, center, flex-end, space-between, space-around, stretch
- **Vertical Align**: Top, middle, bottom, space-between, space-around, stretch
- **Gap**: None, Tiny (0.25), Small (0.5), Medium (1), Large (1.5)
- **Wrap**: Enable/disable item wrapping
- **Flex Factor**: Configuration factor for flexible sizing

### Grid Settings

- **Columns**: Number of grid columns (default: 6)
- **Rows**: Number of grid rows (default: 6)
- **Column Span**: Individual component column spans (default: 1)
- **Row Span**: Individual component row spans (default: 1)

### Tabs Configuration

- **Theme**: Budibase, Buttons, Negative Buttons, or List styles
- **Size**: Small, Medium, Large tab sizes
- **Text Align**: Left, Center, Right alignment
- **Active Tab**: Control which tab is initially active
- **Icons Only**: Display tab icons only
- **Quiet Tabs**: Enable subtle tab styling

### SplitView Configuration

- **Split Ratio**: Define panel size ratios
- **Resizable**: Enable/disable resize handles
- **Min Sizes**: Set minimum panel dimensions

### Collapsible Panel Settings

- **Collapsed**: Start in collapsed state
- **Panel Title**: Title when expanded
- **Collapse Title**: Title when collapsed
- **Collapse Icon**: Icon for collapsed state
- **Collapse Size**: Dimensions of collapsed panel (default: 3rem)
- **Collapse Side**: Left, Top, Right, or Bottom collapse direction

### Field Group Configuration

- **Label Position**: Left or Above positioning
- **Label Width**: Width for left-aligned labels
- **Grid Columns**: Number of form columns (default: 3)
- **Disabled**: Enable/disable entire group

## 📋 Usage Examples

### Dashboard Layout - Grid Mode

Configure a complex dashboard:

- Mode: Grid
- Columns: 12
- Rows: 8
- Child components positioned in different grid areas
- Spacing: Medium gap for visual separation

### Form Layout - Field Group Mode

Create an organized form:

- Mode: Field Group
- Label Position: Left
- Label Width: 6rem
- Grid Columns: 3
- Individual fields configured with appropriate column spans

### Navigation Layout - Tabs Mode

Build a tabbed interface:

- Mode: Tabs
- Theme: Budibase
- Tabs Size: Medium
- Individual tab items configured with titles, icons, and content
- Active Tab: Controlled by binding

### Content Sections - Collapsible Container

Organize content sections:

- Mode: Container
- Collapsible: Enabled
- Collapse Side: Left
- Panel Title: "Section Header"
- Collapse Title: "Section"
- Collapse Icon: ThreeDots

### Sidebar Layout - SplitView Mode

Build a layout with sidebar:

- Mode: SplitView
- Primary Panel: Navigation components
- Secondary Panel: Main content area
- Split Ratio: 25/75

## 🎨 Styling & Theming

### Layout Styling

- **Padding Control**: Adjust container padding
- **Border Options**: Customize border styles
- **Background Colors**: Solid or gradient backgrounds
- **Corner Radius**: Rounded or sharp corners

### Interactive Styling

- **Hover Effects**: Background, border, and text color changes
- **Transition Animations**: Smooth state transitions
- **Focus Indicators**: Accessible focus management
- **Active States**: Visual feedback for user interactions

### Responsive Design

- **Breakpoint Adjustments**: Layout changes at different screen sizes
- **Flexible Components**: Child components that adapt to available space
- **Content Flow**: Automatic layout adjustments for content fitting

## 🔗 Integration

### Data Provider Integration

- **Data-Driven Layouts**: Adjust layout based on data
- **Dynamic Templates**: Template-based configuration
- **Context-Aware Styling**: Styling that responds to data conditions

### Component Nesting

- **Hierarchical Structures**: Nested containers and layouts
- **Parent-Child Communication**: Component interaction through event system
- **Shared Context**: Data context sharing between nested components

### Event System

- **Click Events**: Standard and right-click interactions
- **Tab Changes**: Tab-specific navigation events
- **Panel Interactions**: Expand/collapse state changes
- **Action Integration**: Event-driven action execution

## 📱 Responsiveness & Accessibility

### Responsive Behavior

- **Container Responsiveness**: Adaptive container sizing
- **Grid Fluidity**: Adjustable grid columns and rows
- **Content Reflow**: Efficient content reorganization
- **Scaling Factors**: Consistent sizing across screen sizes

### Accessibility Features

- **Keyboard Navigation**: Comprehensive keyboard traversal
- **Screen Reader Integration**: Inclusive label provision
- **Focus Management**: Logical interaction sequence
- **Semantic Structure**: Properly structured component hierarchy

## 🐛 Troubleshooting

### Layout Issues

- **Flex Problems**: Verify flex factor and sizing configurations
- **Grid Positioning**: Check column and row span settings
- **Alignment Problems**: Confirm direction and alignment settings
- **Spacing Issues**: Adjust gap and padding values appropriately

### Interactive Problems

- **Click Events**: Validate event configuration settings
- **Tab Navigation**: Review tab configuration and theme settings
- **Hover Effects**: Check hover configuration and styling
- **Collision Detection**: Assess component overlapping and interaction

Find out more about developing [Custom Plugins](https://docs.budibase.com/docs/custom-plugin) and [Budibase](https://github.com/Budibase/budibase).
