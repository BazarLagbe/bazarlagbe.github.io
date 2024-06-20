<script>
  import { onMount } from "svelte";

  let products = [];
  let priceUpdatedAt = "";
  let selectedCategory = "";
  let searchQuery = "";

  // Pagination state
  let currentPage = 1;
  let productsPerPage = 10; // Number of products per page

  const fetchProducts = async () => {
    try {
      const response = await fetch("https://raw.githubusercontent.com/BazarLagbe/pricelist/main/products.json");
      const data = await response.json();

      // Extract priceUpdatedAt and products
      priceUpdatedAt = data.priceUpdatedAt;
      priceUpdatedAt = new Date(priceUpdatedAt).toLocaleString("en-US", {
        year: "numeric",
        month: "long",
        day: "numeric",
        hour12: true,
      });
      products = data.products;

      console.log("Fetched products:", products); // Debug log
      console.log("Price updated at:", priceUpdatedAt); // Debug log
    } catch (error) {
      console.error("Error fetching products:", error);
    }
  };

  onMount(fetchProducts);

  // Reactive declaration for categories
  $: categories = [...new Set(products.map((p) => p.category))];
  $: console.log("Categories:", categories); // Debug log

  // Reactive declaration for filtered products
  $: filteredProducts = products.filter((p) => {
    const categoryMatch =
      selectedCategory === "" || p.category === selectedCategory;
    const nameMatch = p.name.toLowerCase().includes(searchQuery.toLowerCase());
    return categoryMatch && nameMatch;
  });

  // Paginated products
  $: paginatedProducts = () => {
    const startIndex = (currentPage - 1) * productsPerPage;
    const endIndex = startIndex + productsPerPage;
    return filteredProducts.slice(startIndex, endIndex);
  };

  // Total pages calculation
  $: totalPages = Math.ceil(filteredProducts.length / productsPerPage);

  const selectCategory = (category) => {
    selectedCategory = category;
    currentPage = 1; // Reset to first page when category changes
    console.log("Selected Category:", selectedCategory); // Debug log
  };

  const nextPage = () => {
    if (currentPage < totalPages) {
      currentPage++;
    }
  };

  const prevPage = () => {
    if (currentPage > 1) {
      currentPage--;
    }
  };

  const handleSearch = () => {
    currentPage = 1; // Reset to first page when search query changes
  };
</script>

<!-- Import Google Fonts -->
<svelte:head>
  <link
    href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap"
    rel="stylesheet"
  />
</svelte:head>

<main>
  <!-- Website Title -->
  <header>
    <h1>Bazar<span class="thin">Lagbe?</span></h1>
  </header>

  <!-- Modern Search Bar -->
  <div class="search-bar">
    <input
      type="text"
      placeholder="Search by product name"
      bind:value={searchQuery}
      on:input={handleSearch}
    />
    <button>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        class="icon icon-tabler icon-tabler-search"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        stroke-width="2"
        stroke="#fff"
        fill="none"
        stroke-linecap="round"
        stroke-linejoin="round"
      >
        <path stroke="none" d="M0 0h24v24H0z" fill="none" />
        <circle cx="10" cy="10" r="7" />
        <line x1="21" y1="21" x2="15" y2="15" />
      </svg>
    </button>
  </div>

  <!-- Category Chips -->
  <div class="category-chips">
    <button
      on:click={() => selectCategory("")}
      class:selected={selectedCategory === ""}>All</button
    >
    {#each categories as category}
      <button
        on:click={() => selectCategory(category)}
        class:selected={selectedCategory === category}>{category}</button
      >
    {/each}
  </div>

  <!-- Product Listing -->
  {#if filteredProducts.length === 0}
    <p>No products found.</p>
  {:else}
    <div class="updatedAt">
      <p>Price list updated at: {priceUpdatedAt}</p>
    </div>
    <table>
      <thead>
        <tr>
          <th>Product</th>
          <th>Price</th>
          <th>Discount</th>
        </tr>
      </thead>
      <tbody>
        {#each paginatedProducts() as product, index}
          <tr class:row-even={index % 2 === 0} class:row-odd={index % 2 !== 0}>
            <td>{product.name} ({product.category})</td>
            <td
              ><del>৳{product.mrp}</del><br /><strong
                >৳{product.discounted_price}</strong
              ></td
            >
            <td>৳{product.mrp - product.discounted_price}</td>
          </tr>
        {/each}
      </tbody>
    </table>

    <!-- Pagination Controls -->
    <div class="pagination">
      <button on:click={prevPage} disabled={currentPage === 1}>Previous</button>
      <span>Page {currentPage} of {totalPages}</span>
      <button on:click={nextPage} disabled={currentPage === totalPages}
        >Next</button
      >
    </div>
  {/if}

  <!-- Copyright Footer -->
  <footer>
    <p>&copy; 2024 Bazar Lagbe? All rights reserved.</p>
  </footer>
</main>

<style>
  body {
    font-family: "Poppins", sans-serif;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    background-color: #f5f5f5;
    color: #333;
    line-height: 1.6;
  }

  header {
    font-family: "Poppins", sans-serif;
    color: #8b0000;
    padding: 2rem 0;
    text-align: center;
    margin-bottom: 2rem;
  }

  h1 {
    margin: 0;
    font-weight: 600;
    font-size: 2.5rem;
  }
  .thin {
    font-weight: 400 !important;
  }

  p {
    font-size: 1.2rem;
    opacity: 0.8;
    margin-top: 0.5rem;
  }

  main {
    max-width: 800px;
    margin: 0 auto;
    padding: 0 1rem;
  }

  /* Modern Search Bar Styles */
  .search-bar {
    display: flex;
    align-items: center;
    margin-bottom: 1rem;
  }

  .search-bar input {
    font-family: "Poppins", sans-serif;
    flex: 1;
    padding: 1rem;
    font-size: 0.8rem;
    border: 2px solid #8b0000; /* Darkish red border */
    border-radius: 0.5rem 0 0 0.5rem;
    outline: none;
    transition: border-color 0.3s ease;
  }

  .search-bar input:focus {
    border-color: #5a0000; /* Darker shade on focus */
  }

  .search-bar button {
    background-color: #8b0000; /* Darkish red background */
    color: white;
    border: none;
    border-radius: 0 0.5rem 0.5rem 0;
    padding: 1rem;
    cursor: pointer;
    transition:
      background-color 0.3s ease,
      color 0.3s ease;
  }

  .search-bar button:hover {
    background-color: #5a0000; /* Darker shade on hover */
  }

  .search-bar svg {
    width: 1.5rem;
    height: 1.5rem;
    vertical-align: middle;
  }

  /* Category Chips and Table Styles */
  .category-chips {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    row-gap: 0.1rem;
    margin-bottom: 1rem;
  }

  .category-chips button {
    padding: 0.5rem 1rem;
    font-family: "Poppins", sans-serif;
    border: none;
    background-color: #8b0000; /* Darkish red background */
    color: white;
    border-radius: 1rem;
    cursor: pointer;
    font-family: "Poppins", sans-serif;
    font-size: 14px;
    transition:
      background-color 0.3s ease,
      color 0.3s ease;
  }

  .category-chips button.selected {
    background-color: #5a0000; /* Darker shade on selected state */
  }

  table {
    font-family: "Poppins", sans-serif;
    width: 100%;
    border-collapse: collapse;
    margin-top: 1rem;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    border-radius: 0.5rem;
    overflow: hidden;
  }

  th,
  td {
    padding: 1rem;
    border-bottom: 1px solid #ddd;
  }

  th {
    background-color: #8b0000; /* Darkish red background */
    color: white;
    font-weight: 600;
    text-align: left;
  }

  tbody tr.row-even {
    background-color: #f9f9f9;
  }

  tbody tr.row-odd {
    background-color: #ffffff;
  }

  /* Pagination Styles */
  .pagination {
    font-family: "Poppins", sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 1rem;
  }

  .pagination button {
    font-family: "Poppins", sans-serif;
    padding: 0.5rem 1rem;
    margin: 0 0.5rem;
    background-color: #8b0000; /* Darkish red background */
    color: white;
    border: none;
    border-radius: 0.5rem;
    cursor: pointer;
    transition:
      background-color 0.3s ease,
      color 0.3s ease;
  }

  .pagination button:disabled {
    background-color: #ccc;
    color: #666;
    cursor: not-allowed;
  }

  .pagination button:hover {
    background-color: #5a0000; /* Darker shade on hover */
  }

  .pagination span {
    margin: 0 1rem;
  }

  .updatedAt p {
    font-family: "Poppins", sans-serif;
    font-size: 12px;
  }

  footer {
    font-family: "Poppins", sans-serif;
    text-align: center;
    padding: 1rem 0;
    padding-top: 40px;
    color: #000;
    width: 100%;
  }
  footer p {
    font-size: 12px;
  }
  footer a {
    color: #8b0000;
  }
</style>
