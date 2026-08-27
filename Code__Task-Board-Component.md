# Task Board Component

#### File : task-board.ts
```ts
import { Component } from '@angular/core';

// Shape of one task. Using a TypeScript type keeps status/priority values fixed
// so we cannot accidentally write a wrong string somewhere.
type TaskStatus = 'pending' | 'in-progress' | 'completed' | 'cancelled';
type TaskPriority = 'low' | 'medium' | 'high';

interface Task {
  id: number;
  title: string;
  assignee: string;
  avatarUrl: string;
  status: TaskStatus;
  priority: TaskPriority;
  dueInDays: number; // negative number means task is already overdue
}

@Component({
  selector: 'app-task-board',
  templateUrl: './task-board.html',
  styleUrl: './task-board.css',
})
export class TaskBoardComponent {
  // Master list of tasks. This is the "source of truth", we never filter this array itself.
  tasks: Task[] = [
    {
      id: 1,
      title: 'Fix login page crash',
      assignee: 'Rishabh',
      avatarUrl: 'https://i.pravatar.cc/40?img=1',
      status: 'in-progress',
      priority: 'high',
      dueInDays: -1,
    },
    {
      id: 2,
      title: 'Design new dashboard UI',
      assignee: 'Aman',
      avatarUrl: 'https://i.pravatar.cc/40?img=2',
      status: 'pending',
      priority: 'medium',
      dueInDays: 3,
    },
    {
      id: 3,
      title: 'Write unit tests for auth service',
      assignee: 'Priya',
      avatarUrl: 'https://i.pravatar.cc/40?img=3',
      status: 'completed',
      priority: 'low',
      dueInDays: 0,
    },
    {
      id: 4,
      title: 'Setup CI/CD pipeline',
      assignee: 'Rohit',
      avatarUrl: 'https://i.pravatar.cc/40?img=4',
      status: 'cancelled',
      priority: 'medium',
      dueInDays: 5,
    },
    {
      id: 5,
      title: 'Refactor payment module',
      assignee: 'Rishabh',
      avatarUrl: 'https://i.pravatar.cc/40?img=5',
      status: 'pending',
      priority: 'high',
      dueInDays: 1,
    },
  ];

  // Simple UI state driven by user input, plain properties (no signal used here).
  statusFilter: 'all' | TaskStatus = 'all';
  searchTerm = '';
  selectedTaskId: number | null = null;
  lastActionLog = 'No action performed yet.';

  // This dropdown list is used with @for to build the filter <select> options.
  statusOptions: Array<'all' | TaskStatus> = [
    'all',
    'pending',
    'in-progress',
    'completed',
    'cancelled',
  ];

  // Getter runs every change detection cycle and returns the tasks that
  // match both the status filter and the search box text.
  get filteredTasks(): Task[] {
    return this.tasks.filter((task) => {
      const matchesStatus = this.statusFilter === 'all' || task.status === this.statusFilter;
      const matchesSearch = task.title.toLowerCase().includes(this.searchTerm.toLowerCase());
      return matchesStatus && matchesSearch;
    });
  }

  // Event binding target: fires on (change) of the status <select>.
  // $event is the native DOM Event, we read the selected value from its target.
  onStatusFilterChange(event: Event): void {
    const selectElement = event.target as HTMLSelectElement;
    this.statusFilter = selectElement.value as 'all' | TaskStatus;
  }

  // Event binding target: fires on (input) of the search box, on every keystroke.
  onSearchInput(event: Event): void {
    const inputElement = event.target as HTMLInputElement;
    this.searchTerm = inputElement.value;
  }

  // Clicking a row selects it, this is used for the [class.selected-row] property binding.
  selectTask(task: Task): void {
    this.selectedTaskId = task.id;
    this.lastActionLog = `Selected task #${task.id} - "${task.title}"`;
  }

  // The "mark complete" button sits inside a clickable row, so we stop the
  // click from bubbling up and triggering selectTask() as well.
  toggleComplete(task: Task, event: Event): void {
    event.stopPropagation();
    task.status = task.status === 'completed' ? 'pending' : 'completed';
    this.lastActionLog = `Task #${task.id} marked as "${task.status}"`;
  }

  // track function for @for. Angular uses the returned id to match items
  // between renders instead of re-creating every row from scratch.
  trackByTaskId(_index: number, task: Task): number {
    return task.id;
  }

  // Used only to demonstrate property binding on a "select all" checkbox.
  get areAllTasksCompleted(): boolean {
    return this.tasks.every((task) => task.status === 'completed');
  }
}

```


#### File : task-board.html
``` HTML
<section class="task-board">
  <h2>Team Task Board</h2>

  <!-- ===================== TOOLBAR ===================== -->
  <div class="toolbar">
    <!-- Event binding: (input) fires on every keystroke, $event carries the native DOM event -->
    <input
      type="text"
      placeholder="Search task by title..."
      class="search-box"
      (input)="onSearchInput($event)"
    />

    <!-- Event binding: (change) fires when a new option is picked -->
    <select class="status-select" (change)="onStatusFilterChange($event)">
      <!-- @for loop to build options from a plain array, track by the value itself since it is a primitive string -->
      @for (option of statusOptions; track option) {
        <option [value]="option">{{ option }}</option>
      }
    </select>
  </div>

  <!-- =============== @if / @else if / @else DEMO =============== -->
  <!-- Shows a banner depending on overall team progress -->
  @if (filteredTasks.length === 0) {
    <p class="hint">Try changing the search text or the status filter above.</p>
  } @else if (areAllTasksCompleted) {
    <p class="banner success">Great job! Every task is completed.</p>
  } @else {
    <p class="banner info">Work is still in progress, keep going.</p>
  }

  <!-- ===================== TASK TABLE ===================== -->
  <table class="task-table">
    <thead>
      <tr>
        <th>#</th>
        <th>Avatar</th>
        <th>Title</th>
        <th>Priority</th>
        <th>Status</th>
        <th>Action</th>
      </tr>
    </thead>
    <tbody>
      <!--
        @for loop over the filtered task list.
        track trackByTaskId(...) tells Angular how to identify each row so it does not
        re-create the whole row when only one task changes.
        We also grab index/first/last/odd/count as local template variables using "let".
      -->
      @for (
        task of filteredTasks;
        track trackByTaskId($index, task);
        let i = $index;
        let isFirst = $first;
        let isLast = $last;
        let isOdd = $odd;
        let total = $count
      ) {
        <!--
          Event binding: (click) on the row calls selectTask(task).
          Property binding: [class.xxx] toggles a CSS class based on a boolean expression.
          isOdd gives zebra striping, isFirst highlights the very first row, isLast adds a thicker border.
        -->
        <tr
          (click)="selectTask(task)"
          [class.selected-row]="selectedTaskId === task.id"
          [class.odd-row]="isOdd"
          [class.first-row]="isFirst"
          [class.last-row]="isLast"
        >
          <!-- i is zero based, so we add 1 to show a normal 1,2,3... serial number -->
          <td>{{ i + 1 }}</td>

          <td>
            <!-- Property binding: [src] and [title] set element attributes/properties from component data -->
            <img
              class="avatar"
              [src]="task.avatarUrl"
              [title]="task.assignee"
              [alt]="task.assignee"
            />
          </td>

          <td>
            {{ task.title }}
            <!-- $first is used again here just to show a "NEW" tag only on the first row -->
            @if (isFirst) {
              <span class="new-tag">NEW</span>
            }
          </td>

          <td>
            <!--
              Nested @if / @else if / @else purely to color-code priority.
              This is INSIDE the @for loop, so it runs once per task.
            -->
            @if (task.priority === 'high') {
              <span class="priority high">High</span>
            } @else if (task.priority === 'medium') {
              <span class="priority medium">Medium</span>
            } @else {
              <span class="priority low">Low</span>
            }
          </td>

          <td>
            <!--
              @switch is used here instead of @if chain because status has a fixed,
              known set of values, this reads cleaner than many @else if blocks.
            -->
            @switch (task.status) {
              @case ('pending') {
                <span class="status-badge pending">Pending</span>
              }
              @case ('in-progress') {
                <span class="status-badge in-progress">In Progress</span>
              }
              @case ('completed') {
                <span class="status-badge completed">Completed</span>
              }
              @case ('cancelled') {
                <span class="status-badge cancelled">Cancelled</span>
              }
              @default {
                <span class="status-badge unknown">Unknown</span>
              }
            }
          </td>

          <td>
            <!--
              Property binding: [disabled] disables the button once the task is cancelled,
              a cancelled task should not be marked complete.
              Event binding: (click) passes both the task and $event, we call
              event.stopPropagation() inside toggleComplete so the row click (selectTask) does not fire too.
            -->
            <button
              type="button"
              [disabled]="task.status === 'cancelled'"
              (click)="toggleComplete(task, $event)"
            >
              {{ task.status === 'completed' ? 'Reopen' : 'Mark Complete' }}
            </button>
          </td>
        </tr>

        <!-- isLast + total (captured from $count) let us print a summary row after the last task -->
        @if (isLast) {
          <tr class="summary-row">
            <td colspan="6">Showing {{ total }} task(s) that match the current filter.</td>
          </tr>
        }
      } @empty {
        <!--
          @empty belongs to the @for block right above it.
          It renders only when the loop's collection (filteredTasks) has zero items,
          this is different from a manual @if check because Angular manages it for us.
        -->
        <tr>
          <td colspan="6" class="empty-state">No tasks found for this search/filter.</td>
        </tr>
      }
    </tbody>
  </table>

  <!-- Shows the last click/action recorded by selectTask() or toggleComplete() -->
  <p class="log-line">Last action: {{ lastActionLog }}</p>
</section>

```

#### File : task-board.css
```css
.task-board {
  font-family: Arial, sans-serif;
  max-width: 900px;
  margin: 20px auto;
  padding: 16px;
}

.toolbar {
  display: flex;
  gap: 12px;
  margin-bottom: 16px;
}

.search-box,
.status-select {
  padding: 8px 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 14px;
}

.banner {
  padding: 10px 14px;
  border-radius: 6px;
  font-weight: 600;
}
.banner.success {
  background-color: #e6f7ec;
  color: #1e7e34;
}
.banner.info {
  background-color: #eef3ff;
  color: #2952cc;
}
.hint {
  color: #777;
  font-style: italic;
}

.task-table {
  width: 100%;
  border-collapse: collapse;
}

.task-table th,
.task-table td {
  padding: 10px;
  border-bottom: 1px solid #eee;
  text-align: left;
}

/* zebra striping driven by $odd */
.odd-row {
  background-color: #fafafa;
}

/* highlight driven by $first */
.first-row {
  box-shadow: inset 3px 0 0 #00a6ff;
}

/* thicker border driven by $last */
.last-row td {
  border-bottom: 2px solid #999;
}

/* selection driven by click event binding */
.selected-row {
  background-color: #eaf4ff;
  box-shadow: inset 3px 0 0 #ff9800;
}

.avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
}

.new-tag {
  margin-left: 6px;
  padding: 2px 6px;
  font-size: 10px;
  background-color: #ff9800;
  color: #fff;
  border-radius: 4px;
}

.priority {
  padding: 3px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
}
.priority.high {
  background-color: #fdecea;
  color: #c62828;
}
.priority.medium {
  background-color: #fff8e1;
  color: #f9a825;
}
.priority.low {
  background-color: #e8f5e9;
  color: #2e7d32;
}

.status-badge {
  padding: 3px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
  text-transform: capitalize;
}
.status-badge.pending {
  background-color: #f0f0f0;
  color: #555;
}
.status-badge.in-progress {
  background-color: #e3f2fd;
  color: #1565c0;
}
.status-badge.completed {
  background-color: #e6f7ec;
  color: #1e7e34;
}
.status-badge.cancelled {
  background-color: #fdecea;
  color: #c62828;
  text-decoration: line-through;
}
.status-badge.unknown {
  background-color: #eee;
  color: #999;
}

button {
  padding: 6px 10px;
  border: 0;
  border-radius: 6px;
  background-color: #3f51b5;
  color: #fff;
  cursor: pointer;
}
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.summary-row td {
  text-align: center;
  font-style: italic;
  color: #666;
  background-color: #fafafa;
}

.empty-state {
  text-align: center;
  padding: 20px;
  color: #999;
}

.log-line {
  margin-top: 12px;
  font-size: 13px;
  color: #555;
}

```


