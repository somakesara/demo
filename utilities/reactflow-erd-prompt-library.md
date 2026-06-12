# ReactFlow ERD Prompt Library for Cursor AI

Ready-to-use prompts for generating Entity Relationship Diagrams using ReactFlow in Cursor AI IDE.

> **Stack assumed:** React 18+, ReactFlow v11 or v12, TypeScript  
> **ReactFlow v12 note:** v12 uses `@xyflow/react` package and `XYFlow` naming. v11 uses `reactflow` package. Specify which version your project uses in every prompt.

---

## Quick Reference

### Section 1 — View Types
| # | Prompt | Use When |
|---|--------|----------|
| 1 | Basic High-Level ERD | First pass, overview only |
| 2 | Detailed ERD | Full schema with columns/types |
| 3 | Toggle View (High-Level ↔ Detailed) | Need both views in one component |
| 4 | Click to Drill Down | Large schemas, 10+ tables |
| 5 | Multi-Schema / Microservices ERD | Multiple bounded contexts |

### Section 2 — Input-Specific Prompts
| # | Prompt | Use When |
|---|--------|----------|
| 6 | From SQL DDL | Have existing SQL scripts |
| 7 | From Plain English | Early design, no schema yet |
| 8 | From JSON Schema | Lightweight schema definition |
| 9 | From Prisma Schema | Prisma ORM projects |
| 10 | From TypeORM Entities | TypeORM / NestJS projects |
| 11 | From Sequelize Models | Sequelize / Express projects |
| 12 | From Drizzle ORM Schema | Drizzle ORM projects |
| 13 | From Django Models | Python / Django projects |
| 14 | From SQLAlchemy Models | Python / FastAPI projects |
| 15 | From Hibernate / Spring Boot | Java Spring Boot projects |
| 16 | From ActiveRecord Models | Ruby on Rails projects |
| 17 | Codebase Scan (Auto-Detect ORM) | Unknown or mixed ORM |

### Section 3 — Layout Modifiers
| # | Prompt | Use When |
|---|--------|----------|
| 18 | Dagre Top-Down Layout | Clean hierarchical arrangement |
| 19 | Dagre Left-Right Layout | Wide schemas, timeline-style |
| 20 | ELK Layered Layout | Complex graphs, 15+ tables |
| 21 | ELK Force Layout | Organic clustering by relationships |

### Section 4 — Feature Add-Ons
| # | Prompt | Use When |
|---|--------|----------|
| 22 | Custom Dark Theme | Branded or dark-mode ERD |
| 23 | Relationship Legend | Presentations, documentation |
| 24 | Mini-Map + Controls | Large diagrams, navigation |
| 25 | Search / Filter Bar | Large schemas, quick lookup |
| 26 | Export to PNG / SVG | Documentation, sharing |
| 27 | Color-Coded Entity Groups | Domain-driven grouping |
| 28 | Zoom to Entity on Click | Quick navigation in large diagrams |
| 29 | Read-Only vs Interactive Toggle | Presentation vs editing mode |

### Section 5 — Delivery Format
| # | Prompt | Use When |
|---|--------|----------|
| 30 | React Component (Vite / CRA) | Existing React project |
| 31 | Self-Contained HTML (No Build) | No React project, single file |

### Section 6 — Full Feature
| # | Prompt | Use When |
|---|--------|----------|
| 32 | Full Feature ERD (All-in-One) | Production-quality output |

### Section 7 — Troubleshooting
| # | Prompt | Use When |
|---|--------|----------|
| 33 | Fix Overlapping Nodes | Nodes render on top of each other |
| 34 | Fix Missing or Wrong Edges | Edges not drawn or connecting wrong nodes |
| 35 | Fix Edges Not Rendering on Load | Edges missing on first render |
| 36 | Fix Node Width Inconsistency | Nodes different widths, misaligned |
| 37 | Fix Performance on Large Graphs | Lag or freeze with 20+ nodes |
| 38 | Iterative Refinement | Updating an existing ERD |

---

## Section 1 — View Types

### 1. Basic High-Level ERD

```
Generate a high-level ERD using ReactFlow [v11: reactflow / v12: @xyflow/react], React 18+.
Show only entity names and relationships between them.
No columns or field details.
Use labeled edges to show relationship type: 1:1, 1:N, M:N.
Use Dagre top-down auto-layout.
No Mermaid, no D3, no other diagramming libraries.

Input schema:
[PASTE YOUR SCHEMA HERE]
```

---

### 2. Detailed ERD

```
Generate a detailed ERD using ReactFlow [v11: reactflow / v12: @xyflow/react], React 18+.
Each entity node must display:
- Entity name as a distinct header row (bold, visually separated)
- All columns listed below the header, one per row
- Each row: column name + data type
- [PK] marker (bold) for primary key columns
- [FK] marker (italic) for foreign key columns
- UNIQUE shown as [U], NOT NULL shown as [NN] inline after type
Use labeled edges with cardinality markers: 1:1, 1:N, M:N.
Use Dagre top-down auto-layout.
No Mermaid, no D3.

Input schema:
[PASTE YOUR SCHEMA HERE]
```

---

### 3. Toggle View (High-Level ↔ Detailed)

```
Create a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, with a toolbar toggle button to switch between:
1. High-level view — entity names and relationship edges only, no columns
2. Detailed view — full columns, data types, [PK], [FK], [U], [NN] constraints

Rules:
- Node positions must be identical in both views (no layout shift on toggle)
- Node height adjusts based on view; width stays fixed
- Toggle button clearly labels current mode: "View: High-Level" or "View: Detailed"
- Edges and cardinality labels remain visible in both views
Use Dagre top-down for initial layout.
No Mermaid, no D3.

Input schema:
[PASTE YOUR SCHEMA HERE]
```

---

### 4. Click to Drill Down (Expand/Collapse per Node)

```
Create a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, where:
- Default view shows only entity names and relationship edges (high-level)
- Clicking any collapsed entity node expands it in-place to show all columns, data types, [PK], [FK]
- Clicking an expanded node collapses it back
- Expanded and collapsed nodes can coexist simultaneously
- Node height adjusts dynamically; edges re-route automatically on resize
- Add "Expand All" and "Collapse All" buttons in the toolbar
- Re-run Dagre layout after expand/collapse to prevent overlaps
No Mermaid, no D3.

Input schema:
[PASTE YOUR SCHEMA HERE]
```

---

### 5. Multi-Schema / Microservices ERD

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, for a microservices or multi-schema system.
Group entities by service/domain using ReactFlow sub-flows or colored background Group nodes.
Show:
- Intra-service relationships: solid edges
- Cross-service relationships: dashed edges with a label showing the service boundary (e.g., "User Service → Order Service")
- Each service/domain group: clearly labeled header, distinct background color per group
- Detailed view with columns, [PK], [FK] per entity
- Toggle between high-level and detailed view
Use ELK layered auto-layout for better cross-group spacing.
No Mermaid, no D3.

Services and entities:
[LIST EACH SERVICE NAME AND ITS ENTITIES/TABLES]
```

---

## Section 2 — Input-Specific Prompts

### 6. From SQL DDL

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, from the following SQL DDL.
Parse and extract:
- All CREATE TABLE statements → entity nodes
- All column definitions → column rows with data types
- PRIMARY KEY → mark [PK], bold
- FOREIGN KEY ... REFERENCES → draw directed edge with cardinality label
- UNIQUE constraint → show [U] inline
- NOT NULL constraint → show [NN] inline
- Composite PKs → mark all PK columns
Detailed view by default with toggle to high-level.
Use Dagre top-down auto-layout.
No Mermaid, no D3.

SQL DDL:
[PASTE SQL DDL HERE]
```

---

### 7. From Plain English

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, from this plain-English description.

Infer and make explicit before generating:
- All entities and their likely attributes
- Sensible data types for each attribute
- id primary key for each entity
- Foreign key columns and join tables for M:N relationships
- Cardinality for each relationship (1:1, 1:N, M:N)

List your inferred schema in a code comment at the top of the file before rendering.
Render as a detailed ERD with toggle between high-level and detailed view.
Use Dagre top-down auto-layout.
No Mermaid, no D3.

Description:
[DESCRIBE YOUR DOMAIN IN PLAIN ENGLISH]
```

---

### 8. From JSON Schema

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, from this JSON schema.

JSON format rules:
- Each top-level key = table name
- Each value = array of column descriptors as strings
- Column descriptor format: "columnName:dataType[:PK][:FK->TargetTable][:U][:NN]"

Extract all FK annotations and draw labeled directed edges with cardinality.
Detailed view by default with toggle to high-level.
Use Dagre top-down auto-layout.
No Mermaid, no D3.

Example:
{
  "users": ["id:int:PK:NN", "email:varchar:U:NN", "created_at:timestamp"],
  "orders": ["id:int:PK:NN", "user_id:int:FK->users:NN", "total:decimal:NN", "status:varchar"]
}

JSON Schema:
[PASTE YOUR JSON SCHEMA HERE]
```

---

### 9. From Prisma Schema

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, from the following Prisma schema.
Parse and extract:
- All model blocks → entity nodes
- All fields with Prisma types → column rows
- @id, @@id → mark [PK]
- @relation(fields: [...], references: [...]) → draw directed edge with cardinality
- Implicit M:N (no explicit join model) → show as annotated M:N edge with a dashed line
- Explicit M:N join models → show as entity nodes
- @unique, @@unique → show [U]
- Enums → display enum name as column type, list enum values in a tooltip comment
Detailed view with toggle to high-level.
Use Dagre top-down auto-layout.
No Mermaid, no D3.

Prisma Schema:
[PASTE schema.prisma CONTENT HERE]
```

---

### 10. From TypeORM Entities

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, by reading all TypeORM entity files in this project.
Identify files decorated with @Entity().
Extract:
- @Column({ type }) → column rows with data types
- @PrimaryGeneratedColumn(), @PrimaryColumn() → mark [PK]
- @ManyToOne, @OneToMany → draw 1:N edge (ManyToOne side holds FK)
- @OneToOne + @JoinColumn → draw 1:1 edge
- @ManyToMany + @JoinTable → draw M:N dashed edge, identify owning entity
- @Unique → show [U] on relevant columns
Detailed view with toggle to high-level.
Use Dagre top-down auto-layout.
No Mermaid, no D3.
```

---

### 11. From Sequelize Models

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, by reading all Sequelize model files in this project.
Look for files using DataTypes definitions or Model.init() calls.
Extract:
- DataTypes.STRING, INTEGER, DATE, etc. → column rows with types
- primaryKey: true → mark [PK]
- Model.belongsTo, hasOne → draw 1:1 or 1:N edge
- Model.hasMany → draw 1:N edge
- Model.belongsToMany → draw M:N dashed edge, show junction table as node
- allowNull: false → show [NN]
- unique: true → show [U]
Detailed view with toggle to high-level.
Use Dagre top-down auto-layout.
No Mermaid, no D3.
```

---

### 12. From Drizzle ORM Schema

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, from the following Drizzle ORM schema file.
Parse and extract:
- pgTable / mysqlTable / sqliteTable declarations → entity nodes
- Column definitions (varchar, int, timestamp, etc.) → column rows with types
- .primaryKey() → mark [PK]
- references(() => table.column) → draw directed FK edge with cardinality
- .notNull() → show [NN]
- .unique() → show [U]
- relations() definitions → confirm cardinality for edges
Detailed view with toggle to high-level.
Use Dagre top-down auto-layout.
No Mermaid, no D3.

Drizzle Schema:
[PASTE YOUR DRIZZLE SCHEMA FILE CONTENT HERE]
```

---

### 13. From Django Models

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, by reading all Django models.py files in this project.
Extract:
- All classes extending models.Model → entity nodes
- All field types (CharField, IntegerField, DateTimeField, BooleanField, etc.) → column rows
- primary_key=True or auto-generated pk → mark [PK]
- ForeignKey → draw 1:N edge; show on_delete as edge tooltip
- OneToOneField → draw 1:1 edge
- ManyToManyField → draw M:N dashed edge; show through model as junction node if defined
- unique=True, unique_together → show [U]
- null=False (default) → show [NN] where explicitly blank=False
Detailed view with toggle to high-level.
Use Dagre top-down auto-layout.
No Mermaid, no D3.
```

---

### 14. From SQLAlchemy Models

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, by reading all SQLAlchemy model files in this project.
Look for classes extending DeclarativeBase, Base, or using declarative_base().
Extract:
- Mapped[type] or Column(Type) declarations → column rows with types
- primary_key=True → mark [PK]
- ForeignKey("table.column") → draw directed edge
- relationship("Model", back_populates=...) → determine and label cardinality
- unique=True → show [U]
- nullable=False → show [NN]
Detailed view with toggle to high-level.
Use Dagre top-down auto-layout.
No Mermaid, no D3.
```

---

### 15. From Hibernate / Spring Boot

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, by reading all JPA/Hibernate entity files in this project.
Look for Java/Kotlin classes annotated with @Entity.
Extract:
- @Column → column rows with types inferred from Java field types
- @Id, @EmbeddedId → mark [PK]
- @ManyToOne + @JoinColumn → draw 1:N edge
- @OneToOne + @JoinColumn → draw 1:1 edge
- @ManyToMany + @JoinTable → draw M:N dashed edge; show join table as node
- @OneToMany (mappedBy) → draw 1:N edge from parent side
- @Column(unique=true) → show [U]
- @Column(nullable=false) → show [NN]
Detailed view with toggle to high-level.
Use Dagre top-down auto-layout.
No Mermaid, no D3.
```

---

### 16. From ActiveRecord Models (Ruby on Rails)

```
Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, by reading all ActiveRecord model files in this project (app/models/*.rb).
Extract:
- All classes extending ApplicationRecord or ActiveRecord::Base → entity nodes
- Infer columns from schema.rb or db/structure.sql if present; otherwise infer from migrations
- belongs_to → draw 1:N edge (FK on this model's table)
- has_one → draw 1:1 edge
- has_many → draw 1:N edge
- has_many :through → draw M:N edge; show join model as node
- has_and_belongs_to_many → draw M:N dashed edge
- Mark [PK] on id column, [FK] on _id columns
Detailed view with toggle to high-level.
Use Dagre top-down auto-layout.
No Mermaid, no D3.
```

---

### 17. Codebase Scan (Auto-Detect ORM)

```
Scan all model and entity files in this project.
Auto-detect the ORM or database framework in use.
Supported: Prisma, TypeORM, Sequelize, Drizzle, Django ORM, SQLAlchemy, Hibernate/JPA, ActiveRecord, Mongoose.

Generate a ReactFlow ERD [v11: reactflow / v12: @xyflow/react], React 18+, with:
- All entities and fields extracted from the detected ORM
- Relationships inferred from FK annotations, decorators, or associations
- [PK], [FK], [U], [NN] markers on columns
- Cardinality labels on all edges (1:1, 1:N, M:N)
- Toggle between high-level and detailed view
- Dagre top-down auto-layout

Before generating, output a comment listing:
- Detected ORM
- List of entities found
- List of relationships found
No Mermaid, no D3.
```

---

## Section 3 — Layout Modifiers

> Append any of these blocks to a base prompt (1–17) to override layout.

### 18. Dagre Top-Down Layout

```
Append to any prompt:

Layout engine: Dagre (@dagrejs/dagre)
Direction: TB (top-to-bottom)
ranksep: 100 (vertical spacing between ranks)
nodesep: 80 (horizontal spacing between nodes)
Re-run layout on initial render using useLayoutEffect.
Call fitView() with padding: 0.2 after layout completes.
```

---

### 19. Dagre Left-Right Layout

```
Append to any prompt:

Layout engine: Dagre (@dagrejs/dagre)
Direction: LR (left-to-right)
ranksep: 120
nodesep: 60
Re-run layout on initial render using useLayoutEffect.
Call fitView() with padding: 0.2 after layout completes.
Use this for wide schemas or when entity hierarchy reads better left-to-right.
```

---

### 20. ELK Layered Layout

```
Append to any prompt:

Layout engine: ELK (elkjs)
Algorithm: layered
elk.direction: DOWN
spacing.nodeNode: 80
spacing.edgeNode: 40
spacing.layerNode: 100
Run layout asynchronously; show a loading state while ELK calculates positions.
Call fitView() after layout resolves.
Prefer over Dagre for schemas with 15+ tables or many cross-relationships.
```

---

### 21. ELK Force Layout

```
Append to any prompt:

Layout engine: ELK (elkjs)
Algorithm: force
Iterations: 300
Use for schemas where organic clustering by relationship density is preferred over strict hierarchy.
Entities with more relationships will cluster together naturally.
Call fitView() after layout resolves.
```

---

## Section 4 — Feature Add-Ons

> Append any of these blocks to a base prompt to add specific features.

### 22. Custom Dark Theme

```
Append to any prompt:

Apply this dark theme:
Canvas:
  background: #1e1e2e
  backgroundVariant: dots, color #313244

Entity node:
  header background: #313244
  header text: #cdd6f4, font-weight: bold, font-size: 13px
  column row background: #1e1e2e
  column row text: #cdd6f4, font-size: 12px
  [PK] row: left border 3px solid #f9e2af, text bold
  [FK] row: left border 3px solid #89b4fa, text italic
  [U] marker: color #a6e3a1
  [NN] marker: color #f38ba8
  node border: 1px solid #45475a, border-radius: 8px
  selected node border: 2px solid #cba6f7
  node min-width: 220px

Edges:
  stroke: #6c7086
  label background: #1e1e2e
  label text: #a6adc8, font-size: 11px
  1:N edge: solid
  M:N edge: dashed (strokeDasharray: 6 3)
  1:1 edge: dotted (strokeDasharray: 2 3)
  edge type: smoothstep or bezier
```

---

### 23. Relationship Legend

```
Append to any prompt:

Add a fixed legend panel anchored to the bottom-right corner of the canvas (not scrollable with the diagram).
Legend content:
- ─────  One-to-Many (1:N)
- - - -  Many-to-Many (M:N)
- · · ·  One-to-One (1:1)
- [PK]   Primary Key
- [FK]   Foreign Key
- [U]    Unique
- [NN]   Not Null
Style legend to match the diagram theme (dark background if using dark theme).
```

---

### 24. Mini-Map + Controls

```
Append to any prompt:

Add ReactFlow <MiniMap> component:
- Position: bottom-right
- Node color function: return #313244 for regular entities, #45475a for junction/join tables
- MiniMap background: #1e1e2e if dark theme

Add ReactFlow <Controls> component:
- Position: bottom-left
- Show: zoom in, zoom out, fit view, lock/unlock pan

Add a "Fit View" button in the main toolbar that calls the fitView() function with padding: 0.2.
```

---

### 25. Search / Filter Bar

```
Append to any prompt:

Add a search input bar in the top toolbar.
Behavior as user types:
- Match against entity names (case-insensitive)
- Match against column names (case-insensitive)
- Matching nodes: highlight with a bright border (e.g., #cba6f7 or #f9e2af)
- Non-matching nodes: reduce opacity to 0.15
- After 300ms debounce, call fitView() scoped to matching nodes only
- If search is empty: restore all nodes to full opacity, remove highlights
Add a clear (×) button inside the search input.
```

---

### 26. Export to PNG / SVG

```
Append to any prompt:

Add an "Export PNG" button in the toolbar.
On click:
- Use html-to-image toPng() on the ReactFlow viewport element
- Capture the full diagram (set width/height to cover all nodes, not just viewport)
- Apply pixelRatio: 2 for high-resolution output
- Trigger browser download with filename: erd-[schemaName]-[YYYY-MM-DD].png
If a toggle view is present, export the currently active view.
```

---

### 27. Color-Coded Entity Groups

```
Append to any prompt:

Group entities by domain and apply a distinct background color per group using ReactFlow background Group nodes (type: 'group').
Each group node sits behind its entity nodes, with a semi-transparent colored background and a domain label at the top.
Assign group membership based on naming conventions or provide the grouping explicitly:
[LIST GROUPS AND WHICH ENTITIES BELONG TO EACH]

Suggested group colors (semi-transparent):
- Group 1: rgba(137, 180, 250, 0.08) — blue
- Group 2: rgba(166, 227, 161, 0.08) — green
- Group 3: rgba(249, 226, 175, 0.08) — yellow
- Group 4: rgba(203, 166, 247, 0.08) — purple
- Group 5: rgba(243, 139, 168, 0.08) — red
```

---

### 28. Zoom to Entity on Click

```
Append to any prompt:

When the user clicks on an entity node header (not to expand/collapse, but a separate icon or double-click):
- Smoothly zoom and pan the canvas to center that node at zoom level 1.5
- Use ReactFlow's setCenter() or fitBounds() with animation duration 500ms
- Add a "Reset View" button in the toolbar that calls fitView() to show all nodes
```

---

### 29. Read-Only vs Interactive Toggle

```
Append to any prompt:

Add a toolbar toggle to switch between:
1. Interactive mode: nodes are draggable, pan/zoom enabled, nodes selectable
2. Read-only (presentation) mode: nodes are not draggable, pan/zoom still enabled, no selection handles, cursor is default

In read-only mode:
- Hide node resize handles
- Disable node click-to-expand if present
- Show a "Presentation Mode" badge in the corner
```

---

## Section 5 — Delivery Format

### 30. React Component (Vite / CRA)

```
Generate a self-contained React component for ReactFlow [v11: reactflow / v12: @xyflow/react], React 18+, TypeScript.
Structure:
- One main ERDDiagram.tsx component
- Separate ErdNode.tsx for custom node rendering
- erdData.ts for nodes and edges data (schema-derived)
- Wrap in ReactFlowProvider in parent component or inside ERDDiagram
- Export ERDDiagram as default export
- Include required CSS import: import 'reactflow/dist/style.css' (v11) or '@xyflow/react/dist/style.css' (v12)

Input schema:
[PASTE YOUR SCHEMA HERE]
```

---

### 31. Self-Contained HTML (No Build Tool)

```
Generate a single self-contained HTML file with a ReactFlow ERD.
Requirements:
- No Vite, no Webpack, no CRA, no node_modules required
- Load via CDN using ES module imports from esm.sh:
    React 18: https://esm.sh/react@18
    ReactDOM 18: https://esm.sh/react-dom@18
    ReactFlow v11: https://esm.sh/reactflow@11
    Dagre: https://esm.sh/@dagrejs/dagre
- Use <script type="module"> for all JS
- All CSS in a <style> tag (include ReactFlow base styles inline)
- Detailed ERD with toggle between high-level and detailed view
- Dagre top-down auto-layout
- Opens and works by double-clicking the file in any modern browser
No Mermaid, no D3.

Input schema:
[PASTE YOUR SCHEMA HERE]
```

---

## Section 6 — Full Feature

### 32. Full Feature ERD (All-in-One)

```
Generate a production-quality ReactFlow ERD using [v11: reactflow / v12: @xyflow/react], React 18+, TypeScript.

── VIEWS ──────────────────────────────────────────────────────────────
- Global toggle button: switch between high-level (entity names only) and detailed (columns + types + [PK] [FK] [U] [NN])
- Per-node expand/collapse: click entity header to toggle columns for that node individually
- Toolbar buttons: "Expand All", "Collapse All", "Reset View"
- Both global toggle and per-node expand/collapse must work independently

── LAYOUT ─────────────────────────────────────────────────────────────
- Initial layout: Dagre, direction TB, ranksep 100, nodesep 80
- Run layout with useLayoutEffect; call fitView(padding: 0.2) after layout
- Toolbar button: "Re-layout" to reset all node positions via Dagre

── NAVIGATION ─────────────────────────────────────────────────────────
- <MiniMap> bottom-right, node colors: #313244 (entity), #45475a (junction)
- <Controls> bottom-left (zoom in/out, fit view, lock)
- Double-click node header: zoom to that node at scale 1.5 (setCenter, 500ms animation)

── SEARCH ─────────────────────────────────────────────────────────────
- Search bar top-toolbar: match entity names and column names
- Matching nodes: bright highlight border (#cba6f7)
- Non-matching nodes: opacity 0.15
- Debounce 300ms; auto fitView to matching nodes
- Clear (×) button

── EXPORT ─────────────────────────────────────────────────────────────
- "Export PNG" button: html-to-image toPng(), pixelRatio 2, full diagram bounds
- Filename: erd-export-[YYYY-MM-DD].png

── STYLING ────────────────────────────────────────────────────────────
- Dark theme:
    Canvas: background #1e1e2e, dot grid #313244
    Node header: background #313244, text #cdd6f4 bold 13px
    Column rows: background #1e1e2e, text #cdd6f4 12px
    [PK] row: left border 3px solid #f9e2af, bold
    [FK] row: left border 3px solid #89b4fa, italic
    [U]: color #a6e3a1  [NN]: color #f38ba8
    Node border: 1px solid #45475a, border-radius 8px
    Selected border: 2px solid #cba6f7, min-width 220px
- Edges: stroke #6c7086, smoothstep, cardinality labels in #a6adc8 11px
- 1:N: solid  |  M:N: dashed (6 3)  |  1:1: dotted (2 3)

── LEGEND ─────────────────────────────────────────────────────────────
- Fixed legend panel bottom-right (above MiniMap):
    ─────  1:N     - - -  M:N     · · ·  1:1
    [PK] Primary Key   [FK] Foreign Key   [U] Unique   [NN] Not Null

── READ-ONLY MODE ─────────────────────────────────────────────────────
- Toolbar toggle: Interactive ↔ Presentation
- Presentation mode: nodes not draggable, selection handles hidden, badge shown

No Mermaid, no D3.

Input schema:
[PASTE SQL DDL / Prisma / Drizzle / JSON / Plain English HERE]
```

---

## Section 7 — Troubleshooting

### 33. Fix Overlapping Nodes

```
The ReactFlow ERD nodes are overlapping. Fix by:
1. Increase Dagre spacing: ranksep: 150, nodesep: 120
2. Ensure Dagre runs inside useLayoutEffect (not useEffect) so DOM dimensions are available
3. Read actual node dimensions using the ReactFlow nodeInternals map before passing to Dagre
4. Set node width/height in Dagre graph explicitly: g.setNode(id, { width, height })
5. After Dagre calculates positions, call setNodes() with updated x/y values
6. Call fitView({ padding: 0.2 }) in the same useLayoutEffect after setNodes()
Do not hardcode node positions — let Dagre calculate all of them.
```

---

### 34. Fix Missing or Wrong Edges

```
Some ReactFlow ERD edges are missing or connecting wrong nodes. Fix by:
1. Log nodes and edges arrays to console before ReactFlow renders — verify source/target IDs
2. Ensure all edge source and target values exactly match node id values (case-sensitive)
3. Standardize node IDs: use lowercase table names with underscores (e.g., "order_items")
4. Confirm FK columns reference the correct parent table name
5. For M:N relationships: ensure the junction table node exists as its own node, and draw two edges (parent → junction, junction → child)
6. Ensure edges array is defined after nodes array
7. Check for duplicate node IDs — ReactFlow silently drops duplicates
```

---

### 35. Fix Edges Not Rendering on Initial Load

```
ReactFlow ERD edges are missing on first render but appear after interacting with the diagram. Fix by:
1. Ensure ReactFlow has finished mounting before passing edges — wrap initial data in a useEffect with empty dependency array
2. Verify the ReactFlow container div has an explicit height (not just height: 100% with no parent height)
3. Call fitView() inside an onInit callback: <ReactFlow onInit={(instance) => instance.fitView()} ...>
4. If using Dagre, ensure layout runs after nodes are registered — use a 0ms setTimeout or useLayoutEffect to delay layout call
5. Check that ReactFlow CSS is imported: import 'reactflow/dist/style.css'
```

---

### 36. Fix Node Width Inconsistency

```
ReactFlow ERD nodes have inconsistent widths, causing misaligned columns. Fix by:
1. Set a fixed min-width on all custom node components: min-width: 220px in CSS
2. In Dagre, set a uniform default width for all nodes: g.setNode(id, { width: 220, height: estimatedHeight })
3. Estimate height dynamically: header (40px) + (number of columns × 28px)
4. Use a consistent monospace or tabular font for column rows so type labels align
5. Apply box-sizing: border-box on node containers to prevent padding from expanding width
```

---

### 37. Fix Performance on Large Graphs (20+ Nodes)

```
The ReactFlow ERD is lagging or freezing with many nodes. Fix by:
1. Add nodesDraggable={false} when not in interactive mode to reduce re-renders
2. Wrap custom node components in React.memo() to prevent unnecessary re-renders
3. Use the ReactFlow onlyRenderVisibleElements prop (set to true) to skip off-screen nodes
4. Memoize nodes and edges arrays with useMemo() — do not recreate them on every render
5. If using ELK, run layout in a Web Worker to avoid blocking the main thread
6. Reduce edge re-renders: memoize edge data and avoid inline object creation in edge props
7. For 50+ nodes, consider adding a viewport-based culling strategy or clustering small entities
```

---

### 38. Iterative Refinement

```
Here is the current ReactFlow ERD component code:
[PASTE FULL CURRENT COMPONENT CODE HERE]

Make only the following changes:
[DESCRIBE SPECIFIC CHANGES — examples:
  - Add a new "payments" table with columns: id, order_id (FK->orders), amount, method, paid_at
  - Change edge style for M:N relationships from solid to dashed
  - Add an export PNG button to the toolbar
  - Fix the user→orders edge which is currently pointing the wrong direction
]

Rules:
- Do not rewrite or restructure the existing component
- Do not change any functionality not mentioned above
- Preserve all existing state, layout, and styling
- Show only the changed sections with clear comments indicating what changed
```

---

## Input Format Reference

| Format | Best For | FK Accuracy | Recommended Prompt |
|--------|----------|-------------|-------------------|
| SQL DDL | Most precise, any DB | Explicit | #6 |
| Prisma Schema | Prisma projects | Explicit | #9 |
| Drizzle ORM | Drizzle projects | Explicit | #12 |
| TypeORM Entities | NestJS projects | Explicit | #10 |
| Sequelize Models | Express/Node projects | Explicit | #11 |
| Django Models | Python/Django projects | Explicit | #13 |
| SQLAlchemy | Python/FastAPI projects | Explicit | #14 |
| Hibernate / JPA | Java Spring Boot | Explicit | #15 |
| ActiveRecord | Ruby on Rails | Explicit | #16 |
| JSON Schema | Lightweight spec | Semi-explicit | #8 |
| Plain English | Early design, no schema | Inferred | #7 |
| Auto-scan | Mixed/unknown ORM | Varies | #17 |

---

## Prompt Composition Guide

Prompts are designed to be combined: **base + layout + features**.

```
Base prompt     → defines what to generate and from what input  (pick 1 from Section 1 or 2)
Layout modifier → controls how nodes are arranged               (pick 1 from Section 3, optional)
Feature add-ons → extend with UI/UX capabilities               (pick any from Section 4)
Delivery format → specifies output structure                    (pick 1 from Section 5, optional)
```

**Example — Prisma schema, ELK layout, dark theme, search, export:**
```
[Prompt #9 — From Prisma Schema]
+
[Prompt #20 — ELK Layered Layout]
+
[Prompt #22 — Custom Dark Theme]
+
[Prompt #25 — Search / Filter Bar]
+
[Prompt #26 — Export to PNG]
```

Paste them together as one message in Cursor chat.

---

## General Tips

- **Specify ReactFlow version explicitly** — v11 (`reactflow`) and v12 (`@xyflow/react`) have different package names and breaking API changes.
- **SQL DDL and ORM schema files are the most reliable inputs** — FKs are unambiguous, no inference needed.
- **For 15+ tables:** use prompt #4 (drill-down) or #5 (multi-schema) to keep the diagram readable.
- **Always include "No Mermaid, no D3"** — Cursor may default to them without this instruction.
- **Dagre vs ELK:** Dagre is faster to set up and sufficient for most schemas. ELK produces better results for complex graphs with 15+ nodes or many crossing edges.
- **Infer schema first:** When using plain English (#7), ask Cursor to print the inferred schema as a comment before generating code — review it for accuracy before proceeding.
- **Use prompt #38 for all iterations** — always paste existing code to prevent Cursor from rewriting the component from scratch.
- **Self-contained HTML (#31)** is useful for sharing ERDs without needing a running React project.
- **Combine the Full Feature prompt (#32) with a specific input prompt (#6–#17)** for the most complete result in a single generation.
