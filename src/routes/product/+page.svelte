<script>
  import { onMount } from "svelte";
  import bg_banner from "$lib/images/bg-search.png";

  import bg_1 from "$lib/images/message/bg-1.svg";
  import bg_2 from "$lib/images/message/bg-2.svg";
  import bg_3 from "$lib/images/message/bg-3.svg";

  import { ToastContainer, FlatToast } from "svelte-toasts";
  import { showToast } from "../../utils/toast";

  import { Img, Button, Search } from "flowbite-svelte";
  import { Dropdown, DropdownItem } from "flowbite-svelte";
  import { ChevronDownSolid, PlusSolid } from "flowbite-svelte-icons";
  import { MicrophoneSolid, SearchOutline } from "flowbite-svelte-icons";
  import axios from "axios";

  // import { page } from "$app/stores";
  import { Pagination } from "flowbite-svelte";
  import {
    ChevronLeftOutline,
    ChevronRightOutline,
  } from "flowbite-svelte-icons";
  import Product from "./product.svelte";

  import { BASE_URL, Product_Category, Grade } from "../../utils/constants";

  onMount(() => {
    axios
      .get(`${BASE_URL}/protected/product/get`)
      .then((response) => {
        console.log(response.data);
        products = response.data;
      })
      .catch((error) => {
        console.log(error);
        console.log(error.response.status);
        if (error.response.status == 404) {
          showToast("Info", error.response.data.message, "info");
        }
      });
  });

  let products = [];
  let indexGrade = -1;
  let indexCategoryProduct = -1;
  let searchText = "";

  // $: activeUrl = $page.url.searchParams.get("page");
  let pages = [
    { name: 1, href: "/components/pagination?page=6" },
    { name: 2, href: "/components/pagination?page=7" },
    { name: 3, href: "/components/pagination?page=8" },
    { name: 4, href: "/components/pagination?page=9" },
    { name: 5, href: "/components/pagination?page=10" },
  ];

  $: {
    pages.forEach((page) => {
      let splitUrl = page.href.split("?");
      let queryString = splitUrl.slice(1).join("?");
      const hrefParams = new URLSearchParams(queryString);
      let hrefValue = hrefParams.get("page");
      // if (hrefValue === activeUrl) {
      //   page.active = true;
      // } else {
      //   page.active = false;
      // }
    });
    pages = pages;
  }

  const previous = () => {
    alert("Previous btn clicked. Make a call to your server to fetch data.");
  };
  const next = () => {
    alert("Next btn clicked. Make a call to your server to fetch data.");
  };

  function handleVoiceBtn() {
    alert("You clicked voice button");
  }

  const handleKeyPress = (event) => {
    if (event.key === "Enter") {
      getProductsBySearch();
    }
  };

  const getProducts = (url) => {
    axios
      .get(url)
      .then((response) => {
        products = response.data;
      })
      .catch((error) => {
        if (error.response.status === 404) {
          products = [];
        }
      });
  };
  const getProductsByCategory = (index) => {
    console.log(index);
    indexCategoryProduct = index;
    indexGrade = -1;
    searchText = "";
    getProducts(
      `${BASE_URL}/protected/product/get?category=${indexCategoryProduct}`
    );
  };

  const getProductsByGrade = (index) => {
    console.log(index);
    indexCategoryProduct = -1;
    indexGrade = index;
    searchText = "";
    let minPrice, maxPrice;
    minPrice = Grade[index - 1];
    maxPrice = Grade[index];
    if (index === 0) {
      minPrice = 0;
    } else if (index === Grade.length) {
      maxPrice = "";
    }
    getProducts(
      `${BASE_URL}/protected/product/get?minPrice=${minPrice}&maxPrice=${maxPrice}`
    );
  };

  const getProductsBySearch = () => {
    searchText = searchText.trim();
    if (searchText.length) {
      indexCategoryProduct = -1;
      indexGrade = -1;
      getProducts(
        `${BASE_URL}/protected/product/get?description=${searchText}`
      );
    }
  };
</script>

<svelte:head>
  <title>DevPro - Product</title>
  <meta name="description" content="Svelte demo app" />
</svelte:head>

<section>
  <div class="pb-16 flex flex-col justify-center align-middle">
    <div class="flex flex-row justify-center">
      <Img src={bg_banner} class="w-full " alt="sample 1" />
    </div>

    <div class="p-4 flex flex-row justify-center mt-6 w-full">
      <form class="flex gap-2 w-full">
        <Search
          size="lg"
          class="flex  items-center"
          placeholder="Search Products.."
          on:keypress={handleKeyPress}
          bind:value={searchText}
        >
          <button type="button" on:click={handleVoiceBtn} class="outline-none">
            <MicrophoneSolid class="w-4 h-4 mr-2" />
          </button>
        </Search>
        <Button
          size="sm"
          class="!bg-primary-1 text-lg"
          on:click={getProductsBySearch}
        >
          <SearchOutline class="w-5 h-5 mr-2 -ml-1" />
          Search
        </Button>
      </form>
    </div>

    <p class="text-2xl font-semibold ml-4">
      {searchText && searchText.trim().length > 0
        ? `Search for "${searchText}"`
        : ""}
    </p>

    <div class="flex justify-between items-center">
      <div class="grid grid-cols-4 max-w-xl my-3 mb-6">
        <div>
          <Button class="bg-primary-1  text-lg">
            Category
            <ChevronDownSolid class="w-3 h-3 ms-2 text-white dark:text-white" />
          </Button>
          <Dropdown>
            {#each Product_Category as item, index}
              <DropdownItem on:click={() => getProductsByCategory(index)}>
                {item}
              </DropdownItem>
            {/each}
          </Dropdown>
        </div>
        <div>
          <Button class="bg-primary-1 text-lg">
            Pricing
            <ChevronDownSolid class="w-3 h-3 ml-2 text-white dark:text-white" />
          </Button>
          <Dropdown>
            {#each Grade as grade, index}
              <DropdownItem on:click={() => getProductsByGrade(index)}>
                Let than {grade}
              </DropdownItem>
            {/each}
            <DropdownItem on:click={() => getProductsByGrade(Grade.length)}>
              Large than {Grade[Grade.length - 1]}
            </DropdownItem>
          </Dropdown>
        </div>
        <div>
          <Button class="bg-primary-1  text-lg"
            >Sort By<ChevronDownSolid
              class="w-3 h-3 ml-2 text-white dark:text-white"
            /></Button
          >
          <Dropdown>
            <DropdownItem>UX/UI designer</DropdownItem>
            <DropdownItem>Web Developement</DropdownItem>
          </Dropdown>
        </div>
        <div>
          <Button class="bg-primary-1  text-lg"
            >Details<ChevronDownSolid
              class="w-3 h-3 ml-2 text-white dark:text-white"
            /></Button
          >
          <Dropdown>
            <DropdownItem>UX/UI designer</DropdownItem>
            <DropdownItem>Web Developement</DropdownItem>
          </Dropdown>
        </div>
      </div>
      <Button href="/post" class="text-xl bg-primary-1">
        Post new Product
        <PlusSolid class="w-3 h-3 ms-2 text-white dark:text-white"></PlusSolid>
      </Button>
    </div>

    {#if products.length != 0}
      <div
        class="grid xl:grid-cols-4 md:grid-cols-3 sm:grid-cols-2 gap-8 place-items-center"
      >
        {#each products as product, index}
          <Product {product} {index} user={product.user.username} />
        {/each}
      </div>

      <div class="flex justify-center mt-5">
        <Pagination
          {pages}
          large
          on:previous={previous}
          on:next={next}
          icon
          normalClass="bg-primary-0 rounded-xl text-xl text-semibold text-primary-1"
        >
          <svelte:fragment slot="prev">
            <span class="sr-only">Previous</span>
            <ChevronLeftOutline class="w-3 h-3" />
          </svelte:fragment>
          <svelte:fragment slot="next">
            <span class="sr-only">Next</span>
            <ChevronRightOutline class="w-3 h-3" />
          </svelte:fragment>
        </Pagination>
      </div>
    {:else}
      <div class="my-16 text-4xl text-center w-full">No Products Found</div>
    {/if}
  </div>

  <ToastContainer let:data>
    <FlatToast {data} />
  </ToastContainer>
</section>

<style>
  section {
    align-items: center;
    color: #ff8a00;
    max-width: 84rem;
    flex: 1;
    display: flex;
    flex-direction: column;
    padding: 1rem;
    width: 100%;

    margin: 0 auto;
    box-sizing: border-box;
    margin-top: 5rem;
  }
</style>
