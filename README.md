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

- **Collapsible Panels**: Expandable/collapsible containers with custom titles and icons
- **Hover Effects**: Interactive styling on mouse over (background, border, text)
- **Event Integration**: Click and right-click event handling
- **Nested Support**: Hierarchical container structures
- **Responsive Design**: Adaptive layouts for different screen sizes
- **Child Component Context**: Provides data context to child components for shared state

### Styling & Theming

- **Custom Styling**: Padding, background, border, and shadow controls
- **Hover States**: Configurable hover effects for interactive feedback
- **Theme Consistency**: Matches Budibase design system
- **CSS Variables**: Dynamic styling via CSS custom properties

## 🎯 Usage Instructions

### Basic Setup

1. Add the SuperContainer component to your screen
2. Select your layout mode (Container, Grid, Tabs, SplitView, or Field Group)
3. Configure layout-specific settings (e.g., direction, columns, tabs)
4. Add child components and configure their properties
5. Customize styling and interaction options

### Layout Modes

#### Container Mode

- **Direction**: Row (horizontal) or Column (vertical)
- **Alignment**: Stretch, center, flex-start, flex-end, space-between, space-around
- **Gap**: Control spacing between child items
- **Wrapping**: Enable/disable flex-wrap for multi-line layouts
- **Flexible Sizing**: Grow/shrink child items based on content and space
- **Collapsible**: Enable/disable with custom title, icon, and collapse side

#### Grid Mode

- **Column Count**: Configurable grid columns
- **Row Count**: Configurable grid rows
- **Item Positioning**: Column and row spans for individual components
- **Responsive Grids**: Adaptive column/row adjustments
- **Gap**: Spacing between grid items

#### Tabs Mode

- **Theme Selection**: Budibase, Buttons, Negative Buttons, List themes
- **Tab Sizing**: Small, Medium, Large options
- **Icon Support**: Icons for each tab
- **Navigation**: Active tab control and change events
- **Quiet Mode**: Subtle tab styling option
- **Alignment**: Control tab alignment (horizontal/vertical)

#### SplitView Mode

- **Resizable Panels**: Drag to adjust panel sizes
- **Ratio Control**: Define initial split ratios
- **Multiple Panels**: Support for complex panel arrangements
- **Smooth Interactions**: Fluid resizing animations
- **Direction**: Row or column-based splits

#### Field Group Mode

- **Label Position**: Left, top, or other positions
- **Label Width**: Configurable width for labels
- **Grid Columns**: Number of columns for form fields
- **Disabled State**: Enable/disable the entire group

### Collapsible Features

- **Collapse Side**: Left, right, top, or bottom
- **Collapse Size**: Minimum size when collapsed
- **Panel Title**: Title for the expanded state
- **Collapse Title**: Title for the collapsed state
- **Collapse Icon**: Custom icon for the expander

## 🔧 Configuration Properties

### Layout Mode Selection

- `mode`: Layout mode ('container', 'grid', 'tabs', 'splitview', 'fieldgroup')

### Container Layout Settings

- `direction`: Flex direction ('row', 'column')
- `hAlign`: Horizontal alignment ('stretch', 'center', 'flex-start', 'flex-end', 'space-between', 'space-around')
- `vAlign`: Vertical alignment ('stretch', 'center', 'flex-start', 'flex-end', 'space-between', 'space-around')
- `gap`: Spacing between items (rem)
- `wrap`: Enable flex wrapping (boolean)

### Grid Settings

- `gridColumns`: Number of grid columns
- `gridRows`: Number of grid rows
- `gap`: Grid gap (rem)

### Tabs Configuration

- `theme`: Tab theme ('budibase', 'buttons', 'negative-buttons', 'list')
- `tabsAlignment`: Tab alignment
- `quietTabs`: Enable quiet tab styling (boolean)
- `activeTab`: Index of the initially active tab
- `tabsIconsOnly`: Show only icons in tabs (boolean)
- `list_title`: Title for list theme
- `list_icon`: Icon for list theme

### SplitView Configuration

- `direction`: Split direction ('row', 'column')

### Collapsible Panel Settings

- `collapsible`: Enable collapsible functionality (boolean)
- `collapsed`: Initial collapsed state (boolean)
- `collapseSide`: Side for collapse ('left', 'right', 'top', 'bottom')
- `collapseSize`: Size when collapsed (rem)
- `panelTitle`: Title for expanded panel
- `collapseTitle`: Title for collapsed panel
- `collapseIcon`: Icon for collapse toggle

### Field Group Configuration

- `labelPos`: Label position ('left', 'top')
- `labelWidth`: Width of labels (rem)
- `disabled`: Disable the field group (boolean)

### Child Item Options

- `flex`: Flex behavior ('grow', 'shrink')
- `flexFactor`: Flex factor for sizing
- `colSpan`: Column span for grid items
- `rowSpan`: Row span for grid items
- `title`: Title for tab items
- `icon`: Icon for tab items
- `color`: Color for tab items
- `tabDisabled`: Disable specific tab (boolean)
- `isTabSection`: Mark as tab section (boolean)

### Styling Options

- `hoverBackground`: Background color on hover
- `hoverBorder`: Border color on hover
- `hoverText`: Text color on hover

### Events

- `onTabChange`: Triggered on tab change
- `onClick`: Triggered on container click
- `onRightClick`: Triggered on right-click
- `onShow`: Triggered when child is shown

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

### Layout Customization

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
- **Content Reflow**: Automatic layout adjustments for content fitting

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
- **Screen Reader Integration**: Proper label provision
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
- **Collapse Detection**: Assess component overlapping and interaction

Find out more about developing [Custom Plugins](https://docs.budibase.com/docs/custom-plugin) and [Budibase](https://github.com/Budibase/budibase).
