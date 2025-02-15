<!-- File: docs/PRD.md -->
## Features

*These features extend the existing [frappe-gantt](https://github.com/frappe/gantt) functionality.*

### Bar2

- **Overview**  
  - Introduce an optional secondary bar (`bar2`) for every task.  
  - Rendered behind the main task bar, at the same height but shifted upward slightly (via `offset_y`) so its full length is visible when stacked.
  - Behaves similarly to the main task bar in terms of configuration and error handling (e.g., if configuration is incomplete, throw an error).

- **Task Configuration**  

  **Mapping of properties:**

  | **Bar2 Property**       | **Equivalent Bar Property** |
  | ----------------------- | --------------------------- |
  | `bar2_start`            | `start`                     |
  | `bar2_end`              | `end`                       |
  | `bar2_duration`         | `duration`                  |
  | `bar2_custom_class`     | `custom_class`              |

  **Common/shared with:**
  - `name`, `id`, `dependencies`, `important`

  **Not shared with bar:**
  - `progress`  (only if bar2_show_progress is true)

  **Bar2-specific:**
  - `bar2_show_progress` – if true, display progress in bar2 just like the main bar. default is false. 

- **Shared Dependency:**  
  - Bar2 uses the same `dependencies` keyword. If a task has dependencies, both bar and bar2 will reflect them (provided bar2 is enabled).

---

#### Implementation

- **Module Structure & Separation**  
  - **New files:**  
    Create separate modules: `bar2.js`, `bar2_arrow.js`, and `bar2_index.js`.  
    This approach keeps the original `bar.js` and `arrow.js` intact, allowing minimal changes for integration and easier merges with future updates.
  
- **Reusing Existing Bar Functions**  
  - In `bar2.js`, directly import and call functions from `bar.js` (or its helpers) to avoid duplicating logic.  
  - *Example:*
    ```javascript:src/bar2.js
    import { someHelperFunction } from './bar';
    
    function drawBar2(parameters) {
        // Reuse calculation or drawing logic from bar.js
        const result = someHelperFunction(parameters);
        // then continue with bar2-specific adjustments
    }
    ```
  - This ensures that you reuse what’s already offered without abstracting shared logic into a common utility—thus leaving `bar.js` largely unchanged.

- **Integration Points**  
  - Use minimal hooks in the original flow to detect if `bar2_options.enabled` is true.  
  - When a task is updated, if bar2 is enabled, the corresponding functions in `bar2_index.js` are called.  
  - This decoupled approach lets bar2 remain independent while keeping changes to core files minimal.

- **DEFAULT_OPTIONS Enhancements (Configuration in `src/defaults.js`)**

  - Shared configuration values (like `arrow_curve`, `bar_corner_radius`, etc.) apply to both bars.
  - Introduce nested configurations for bar2:
  
    - **`bar2_options`** (for task bar properties):
  
      ```javascript:src/defaults.js
      const DEFAULT_BAR2_OPTIONS = {
          enabled: true,  // Bar2 enabled by default
          height: 25,     // Bar2's height (can be adjusted as needed)
          offset_y: -5,   // Vertical offset relative to the main bar
      };
      ```

    - **`bar2_arrow_options`** (for dependency arrow styling):
  
      ```javascript:src/defaults.js
      const DEFAULT_BAR2_ARROW_OPTIONS = {
          color: 'grey',  // Bar2 arrows are rendered in grey
          // Additional options as required
      };
      ```

    - Merge these into the main configuration:

      ```javascript:src/defaults.js
      const DEFAULT_OPTIONS = {
          arrow_curve: 5,
          bar_corner_radius: 3,
          bar_height: 30,
          // ... other common options,
          bar2_options: DEFAULT_BAR2_OPTIONS,
          bar2_arrow_options: DEFAULT_BAR2_ARROW_OPTIONS,
          // ... remaining options
      };
      
      export { DEFAULT_OPTIONS, DEFAULT_VIEW_MODES };
      ```

- **Bar2 Arrow Flow**  
  - Implement the dependency arrows for bar2 in `bar2_arrow.js` much like the existing `arrow.js`, with slight differences (for example, using grey color).  
  - In this file, use bar2-specific DOM references (e.g., `this.from_task.$bar2`) and pull configuration from `bar2_arrow_options`.

- **Mirroring the Original**  
  - Wherever possible, the implementation in bar2 modules should mirror how the original bars and arrows are handled.  
  - Reuse available utilities as much as possible, such as date formats from `date_utils.js` without importing new libraries.

