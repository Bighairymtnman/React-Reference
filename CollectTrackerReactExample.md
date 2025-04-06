# CollectTracker React Hooks & Patterns Examples

This document analyzes real-world React hooks and patterns from the CollectTracker application, demonstrating practical implementations of React concepts.

**Project Repository:** [Link to CollectTracker Repository]

## Table of Contents

- [Form Handling with Hooks](#form-handling-with-hooks)
- [List Rendering Patterns](#list-rendering-patterns)
- [Modal Implementation](#modal-implementation)
- [Search and Filter Functionality](#search-and-filter-functionality)
- [Component Structure and Routing](#component-structure-and-routing)

## Form Handling with Hooks

**File:** [client/src/components/AddCollectionForm.js]([Link to file](https://github.com/Bighairymtnman/CollectTracker/blob/main/client/src/components/AddCollectionForm.js))

The `AddCollectionForm` component demonstrates several important React patterns for form handling:

```jsx
import React, { useState } from "react";
import "./AddCollectionForm.css";
import API_BASE_URL from "../config";

const AddCollectionForm = ({ onCollectionAdded }) => {
  // State Management
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [formData, setFormData] = useState({
    name: "",
    type: "generic",
    customType: "",
  });

  // Collection types for dropdown
  const collectionTypes = [
    { value: "generic", label: "Generic" },
    { value: "books", label: "Books" },
    { value: "records", label: "Records" },
    { value: "custom", label: "Custom" },
  ];

  // Form submission handler
  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const submitData = {
        name: formData.name,
        type: formData.type === "custom" ? formData.customType : formData.type,
      };

      const response = await fetch(`${API_BASE_URL}/collections`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(submitData),
      });
      const data = await response.json();

      // Notify parent component and reset form
      onCollectionAdded(data);
      setIsModalOpen(false);
      setFormData({ name: "", type: "generic", customType: "" });
    } catch (error) {
      console.error("Error adding collection:", error);
    }
  };

  // Conditional rendering for the modal and custom type field
  return (
    <div className="add-collection-container">
      <button className="add-button" onClick={() => setIsModalOpen(true)}>
        + Add Collection
      </button>

      {isModalOpen && (
        <div className="modal-overlay">
          <div className="modal">
            {/* Form content */}
            <form onSubmit={handleSubmit}>
              {/* Form fields */}

              {/* Conditional rendering example */}
              {formData.type === "custom" && (
                <div className="form-group">
                  <label>Custom Type Name:</label>
                  <input
                    type="text"
                    value={formData.customType}
                    onChange={(e) =>
                      setFormData({ ...formData, customType: e.target.value })
                    }
                    required
                  />
                </div>
              )}

              {/* Form buttons */}
            </form>
          </div>
        </div>
      )}
    </div>
  );
};
```

### Key React Hooks & Patterns Demonstrated:

1. **useState Hook for Form State Management**:

   - Uses a single state object (`formData`) to manage multiple form fields
   - Initializes form with default values
   - Demonstrates proper state updates with the spread operator to preserve other field values

2. **Controlled Components Pattern**:

   - Form inputs are fully controlled by React state
   - Each input's value comes from state and is updated through onChange handlers
   - This ensures React is the "single source of truth" for form data

3. **Conditional Rendering**:

   - Modal dialog only renders when `isModalOpen` state is true
   - Custom type input field only appears when 'custom' type is selected
   - Uses the logical AND (`&&`) pattern for conditional rendering

4. **Form Submission with Async/Await**:

   - Prevents default form submission behavior
   - Uses async/await for clean API interaction
   - Properly handles form submission and resets form state afterward

5. **Prop Callbacks for Parent Communication**:

   - Uses the `onCollectionAdded` callback to notify parent component of changes
   - Demonstrates the unidirectional data flow pattern in React

6. **Event Handler Patterns**:

   - Inline arrow functions for simple state updates
   - Separate handler function for complex operations (form submission)
   - Proper event object usage with `preventDefault()`

7. **State Reset Pattern**:
   - Resets form to initial state after successful submission
   - Closes modal upon completion

## List Rendering Patterns

**File:** [client/src/components/CollectionList.js]([Link to file](https://github.com/Bighairymtnman/CollectTracker/blob/main/client/src/components/CollectionList.js))

The `CollectionList` component demonstrates advanced patterns for rendering lists with filtering, sorting, and CRUD operations:

```jsx
import React, { useState, useEffect } from "react";
import { useNavigate } from "react-router-dom";
import "./CollectionList.css";
import API_BASE_URL from "../config";

const CollectionList = () => {
  const navigate = useNavigate();

  // Multiple state variables for different concerns
  const [collections, setCollections] = useState([]);
  const [filteredCollections, setFilteredCollections] = useState([]);
  const [searchTerm, setSearchTerm] = useState("");
  const [sortOption, setSortOption] = useState("name-asc");
  const [editingCollection, setEditingCollection] = useState(null);
  const [showDeleteModal, setShowDeleteModal] = useState(false);
  const [showAddModal, setShowAddModal] = useState(false);
  const [collectionToDelete, setCollectionToDelete] = useState(null);

  // Data fetching with useEffect
  useEffect(() => {
    fetchCollections();
  }, []);

  const fetchCollections = async () => {
    try {
      const response = await fetch(`${API_BASE_URL}/collections`);
      const data = await response.json();
      setCollections(data);
    } catch (error) {
      console.error("Error fetching collections:", error);
    }
  };

  // Derived state with useEffect for filtering and sorting
  useEffect(() => {
    const filtered = Array.isArray(collections)
      ? collections.filter(
          (collection) =>
            collection.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
            collection.type.toLowerCase().includes(searchTerm.toLowerCase())
        )
      : [];
    const sortedAndFiltered = getSortedCollections(filtered);
    setFilteredCollections(sortedAndFiltered);
  }, [searchTerm, collections, sortOption]);

  // Pure function for sorting collections
  const getSortedCollections = (collections) => {
    const sorted = [...collections];
    switch (sortOption) {
      case "name-asc":
        return sorted.sort((a, b) => a.name.localeCompare(b.name));
      case "name-desc":
        return sorted.sort((a, b) => b.name.localeCompare(a.name));
      case "newest":
        return sorted.sort(
          (a, b) => new Date(b.created_at) - new Date(a.created_at)
        );
      case "oldest":
        return sorted.sort(
          (a, b) => new Date(a.created_at) - new Date(b.created_at)
        );
      case "most-items":
        return sorted.sort((a, b) => b.item_count - a.item_count);
      case "least-items":
        return sorted.sort((a, b) => a.item_count - b.item_count);
      default:
        return sorted;
    }
  };

  // JSX with list rendering and conditional content
  return (
    <div className="collections-container">
      {/* Search and sort controls */}
      <div className="search-and-sort">
        <input
          type="text"
          placeholder="Search collections..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
        <select
          value={sortOption}
          onChange={(e) => setSortOption(e.target.value)}
        >
          <option value="name-asc">Name (A-Z)</option>
          <option value="name-desc">Name (Z-A)</option>
          {/* Additional options */}
        </select>
      </div>

      {/* Collections grid with map function */}
      <div className="collections-grid">
        {filteredCollections.map((collection) => (
          <div
            key={collection.id}
            className="collection-card"
            onClick={() => handleCollectionClick(collection.id)}
          >
            <div className="card-header">
              {editingCollection?.id === collection.id ? (
                // Edit mode with inline form
                <form
                  onSubmit={handleSaveEdit}
                  onClick={(e) => e.stopPropagation()}
                >
                  <input
                    value={editingCollection.name}
                    onChange={(e) =>
                      setEditingCollection({
                        ...editingCollection,
                        name: e.target.value,
                      })
                    }
                    autoFocus
                  />
                  <button type="submit">Save</button>
                  <button
                    type="button"
                    onClick={() => setEditingCollection(null)}
                  >
                    Cancel
                  </button>
                </form>
              ) : (
                // Display mode
                <>
                  <h3>{collection.name}</h3>
                  <div
                    className="card-actions"
                    onClick={(e) => e.stopPropagation()}
                  >
                    <button onClick={(e) => handleEdit(e, collection)}>
                      Edit
                    </button>
                    <button
                      onClick={() => {
                        setCollectionToDelete(collection);
                        setShowDeleteModal(true);
                      }}
                    >
                      Delete
                    </button>
                  </div>
                </>
              )}
            </div>
          </div>
        ))}

        {/* Static "Add New" card */}
        <div
          className="collection-card add-card"
          onClick={() => setShowAddModal(true)}
        >
          <div className="add-icon">+</div>
          <p>Add New Collection</p>
        </div>
      </div>

      {/* Modals for add and delete operations */}
    </div>
  );
};
```

### Key React Hooks & Patterns Demonstrated:

1. **Multiple useState Hooks for Complex State Management**:

   - Separates concerns with multiple state variables
   - Uses appropriate state types for different data (arrays, booleans, objects)
   - Demonstrates when to use separate state variables vs. combined objects

2. **useEffect for Data Fetching**:

   - Fetches data on component mount with empty dependency array
   - Properly separates the fetch logic into a named function
   - Handles errors appropriately

3. **useEffect for Derived State**:

   - Calculates filtered and sorted collections when dependencies change
   - Avoids unnecessary state updates with proper dependency array
   - Demonstrates the "computed properties" pattern in React

4. **List Rendering with Keys**:

   - Uses the map function to render collection cards
   - Properly assigns unique keys to list items
   - Demonstrates efficient list rendering

5. **Conditional Rendering Patterns**:

   - Ternary operator for edit/display mode switching
   - Logical AND (`&&`) for modal rendering
   - Fragment syntax (`<>...</>`) for grouping elements without extra DOM nodes

6. **Event Propagation Control**:

   - Uses `e.stopPropagation()` to prevent click events from bubbling
   - Demonstrates understanding of React's event system
   - Prevents unintended navigation when clicking edit/delete buttons

7. **Optimistic UI Updates**:

   - Updates the UI immediately after user actions
   - Maintains consistency between server and client state
   - Provides immediate feedback to user actions

8. **Pure Function Pattern**:

   - `getSortedCollections` is a pure function that doesn't depend on component state
   - Makes the sorting logic testable and reusable
   - Separates data transformation from rendering logic

9. **Routing Integration with useNavigate**:

   - Uses React Router's `useNavigate` hook for programmatic navigation
   - Demonstrates integration between component state and routing

10. **Inline Edit Pattern**:
    - Allows editing collection names directly in the list
    - Toggles between display and edit modes
    - Maintains edit state separately from the main data

This component demonstrates advanced React patterns for list management, including filtering, sorting, and inline editing, while maintaining clean separation of concerns.

## Modal Implementation

**File:** [client/src/components/PhotoGalleryModal.js]([Link to file](https://github.com/Bighairymtnman/CollectTracker/blob/main/client/src/components/PhotoGalleryModal.js))

The `PhotoGalleryModal` component demonstrates advanced React patterns for modal interfaces and image handling:

```jsx
import React, { useState, useEffect, useCallback } from "react";

const PhotoGalleryModal = ({ item, onClose, onRefresh }) => {
  // State management for photos and UI
  const [photos, setPhotos] = useState([]);
  const [currentPhotoIndex, setCurrentPhotoIndex] = useState(0);
  const [isUploading, setIsUploading] = useState(false);

  // Fetch photos with useCallback
  const fetchPhotos = useCallback(async () => {
    try {
      const response = await fetch(
        `${API_BASE_URL}/collections/items/${item.id}/photos`
      );
      if (response.ok) {
        const data = await response.json();
        setPhotos(data);
      }
    } catch (error) {
      console.error("Error fetching photos:", error);
    }
  }, [item.id]);

  // Initialize photos on component mount
  useEffect(() => {
    fetchPhotos();
  }, [fetchPhotos]);

  // Photo navigation with circular array pattern
  const nextPhoto = () => {
    setCurrentPhotoIndex((prev) => (prev + 1) % photos.length);
  };

  const previousPhoto = () => {
    setCurrentPhotoIndex((prev) => (prev - 1 + photos.length) % photos.length);
  };

  return (
    <div className="photo-gallery-modal-overlay">
      <div className="photo-gallery-modal">
        {/* Modal content with navigation */}
        <div className="gallery-content">
          {photos.length > 0 ? (
            <div className="gallery-viewer">
              {/* Navigation buttons and current photo */}
            </div>
          ) : (
            <div className="no-photos">
              <p>No photos in gallery</p>
            </div>
          )}
        </div>
      </div>
    </div>
  );
};
```

### Key React Hooks & Patterns Demonstrated:

1. **useCallback for Memoized Functions**:

   - Uses `useCallback` to memoize the `fetchPhotos` function
   - Prevents unnecessary re-creation of the function on re-renders
   - Properly specifies dependencies (`item.id`) to update when needed

2. **useEffect with Memoized Callback**:

   - Uses the memoized `fetchPhotos` function in the dependency array
   - Ensures the effect only runs when the callback changes
   - Demonstrates proper dependency management

3. **Modal Pattern with Overlay**:

   - Creates a modal dialog with background overlay
   - Provides clear visual separation from the main content
   - Includes proper close button and header

4. **Carousel/Gallery Pattern**:

   - Implements next/previous navigation for photos
   - Uses modulo arithmetic for circular navigation
   - Handles edge cases (no photos, single photo)

5. **Loading State Management**:

   - Tracks upload state with `isUploading`
   - Provides visual feedback during async operations
   - Disables controls during uploads to prevent concurrent operations

6. **Conditional Rendering for Empty States**:

   - Shows different UI when no photos are available
   - Provides clear user feedback for empty states
   - Maintains good UX in all scenarios

7. **File Upload Pattern**:

   - Handles multiple file uploads
   - Processes files sequentially with async/await
   - Updates UI after successful uploads

8. **Index Management After Deletions**:

   - Intelligently adjusts the current photo index after deletion
   - Prevents out-of-bounds errors when deleting the last photo
   - Maintains a good user experience during destructive operations

9. **Props for Component Communication**:
   - Uses `onClose` for modal dismissal
   - Uses `onRefresh` to notify parent of data changes
   - Demonstrates proper component communication patterns

This component showcases advanced React patterns for building interactive modal interfaces with image handling capabilities, proper state management, and optimized rendering.

## Search and Filter Functionality

**File:** [client/src/components/SearchSortControls.js]([Link to file](https://github.com/Bighairymtnman/CollectTracker/blob/main/client/src/components/SearchSortControls.js))

The `SearchSortControls` component demonstrates React patterns for creating reusable, controlled components:

```jsx
import React from "react";
import "./SearchSortControls.css";

const SearchSortControls = ({
  searchTerm,
  onSearchChange,
  sortOption,
  onSortChange,
  sortOptions,
  searchPlaceholder,
}) => {
  return (
    <div className="search-and-sort">
      {/* Search input section */}
      <div className="search-container">
        <input
          type="text"
          placeholder={searchPlaceholder}
          value={searchTerm}
          onChange={(e) => onSearchChange(e.target.value)}
          className="search-input"
        />
      </div>
      {/* Sort selection dropdown */}
      <div className="sort-container">
        <select
          value={sortOption}
          onChange={(e) => onSortChange(e.target.value)}
          className="sort-select"
        >
          {sortOptions.map((option) => (
            <option key={option.value} value={option.value}>
              {option.label}
            </option>
          ))}
        </select>
      </div>
    </div>
  );
};
```

### Key React Patterns Demonstrated:

1. **Controlled Component Pattern**:

   - Both search input and sort select are fully controlled components
   - Values come from props and changes are handled by callback props
   - Maintains the parent component as the single source of truth

2. **Presentational Component Pattern**:

   - Component focuses purely on UI rendering, not business logic
   - All data and behavior are passed through props
   - Makes the component highly reusable across different contexts

3. **Prop Destructuring**:

   - Uses JavaScript destructuring for clean prop access
   - Makes the component signature clear and readable
   - Provides visual documentation of required props

4. **Callback Props Pattern**:

   - Uses function props (`onSearchChange`, `onSortChange`) for communication
   - Passes data up to parent components
   - Follows React's unidirectional data flow principle

5. **Configurable Component Pattern**:

   - Accepts customization props like `searchPlaceholder`
   - Takes dynamic data like `sortOptions` to render different options
   - Maintains flexibility while providing a consistent interface

6. **Event Handler Delegation**:

   - Delegates event handling to parent components
   - Processes event data before passing to callbacks (`e.target.value`)
   - Keeps the component focused on its presentation role

7. **List Rendering with Keys**:
   - Uses the `map` function with proper keys for option rendering
   - Ensures efficient updates when options change
   - Follows React best practices for list rendering

This component demonstrates how to create flexible, reusable UI components that can be composed into larger applications. It separates presentation from logic and provides a clean interface for parent components to control its behavior.

## Component Structure and Routing

**File:** [client/src/App.js]([Link to file](https://github.com/Bighairymtnman/CollectTracker/blob/main/client/src/App.js))

The `App` component demonstrates React patterns for application structure and routing:

```jsx
import React from "react";
import { BrowserRouter as Router, Routes, Route } from "react-router-dom";
import "./App.css";
import MainLayout from "./components/MainLayout";
import CollectionPage from "./components/CollectionPage";

function App() {
  return (
    <Router>
      <div className="App">
        {/* Application Header */}
        <header className="App-header">
          <h1>CollectTracker</h1>
        </header>

        {/* Route Definitions */}
        <Routes>
          <Route path="/" element={<MainLayout />} />
          <Route path="/collection/:id" element={<CollectionPage />} />
        </Routes>
      </div>
    </Router>
  );
}

export default App;
```

### Key React Patterns Demonstrated:

1. **React Router Integration**:

   - Uses React Router v6 for declarative routing
   - Wraps the application in a `Router` component
   - Defines routes with the `Routes` and `Route` components

2. **Route Parameter Pattern**:

   - Uses dynamic route parameters (`:id`) for collection pages
   - Enables navigation to specific resources
   - Follows RESTful URL structure

3. **Component Composition**:

   - Composes the application from smaller, focused components
   - Separates concerns between routing and content
   - Maintains a clean component hierarchy

4. **Declarative UI Structure**:

   - Uses JSX to declaratively define the application structure
   - Clearly separates header and content areas
   - Makes the component hierarchy easy to understand

5. **Consistent Layout Pattern**:

   - Maintains consistent header across all routes
   - Provides a container for route-specific content
   - Creates a cohesive user experience

6. **Function Component Pattern**:
   - Uses modern function component syntax
   - Avoids unnecessary class components
   - Follows React best practices for component definition

This component demonstrates how to structure a React application with routing, providing a solid foundation for navigation between different views while maintaining a consistent user interface.
