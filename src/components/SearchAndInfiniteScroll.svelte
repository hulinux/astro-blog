<script>
  import { onMount } from "svelte"
  import { fade } from "svelte/transition"
  let data = []
  let searchTerm = ""
  let filteredData = []
  let displayedData = []
  let itemsPerLoad = 50
  let loading = false
  let containerRef

  $: {
    filteredData = data.filter((item) =>
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    )
    displayedData = filteredData.slice(0, itemsPerLoad)
  }

  function loadMore() {
    if (loading) return
    loading = true
    setTimeout(() => {
      const currentLength = displayedData.length
      const moreData = filteredData.slice(
        currentLength,
        currentLength + itemsPerLoad
      )
      displayedData = [...displayedData, ...moreData]
      loading = false
    }, 500) // 模拟加载延迟
  }

  function handleScroll() {
    if (!containerRef) return

    const { scrollTop, scrollHeight, clientHeight } = containerRef
    const bottom = scrollHeight - scrollTop <= clientHeight + 100

    if (bottom && !loading && displayedData.length < filteredData.length) {
      loadMore()
    }
  }

  onMount(async () => {
    const response = await fetch("/fonts.json")
    data = await response.json()
    if (containerRef) {
      containerRef.addEventListener("scroll", handleScroll)
    }
    return () => {
      if (containerRef) {
        containerRef.removeEventListener("scroll", handleScroll)
      }
    }
  })
</script>

<div class="container mx-auto p-4">
  <div class="flex items-center mb-4">
    <input
      type="text"
      bind:value={searchTerm}
      placeholder="Search..."
      class="input input-bordered w-full max-w-xs"
    />
    <span class="ml-4">
      {displayedData.length} / {filteredData.length}
    </span>
  </div>
  <div bind:this={containerRef} class="h-[calc(100vh-200px)] overflow-y-auto">
    <div
      class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 2xl:grid-cols-6 gap-4"
    >
      {#each displayedData as item}
        <div class="card card-compact bg-base-100 shadow-xl">
          <div class="card-body">
            <div class="join">
              <span class="card-title text-sm">{item.name}</span>
              <p>{item.description}</p>
              <div class="card-actions justify-end">
                <a
                  href={item.path}
                  class="tooltip"
                  data-tip="download this font"
                  ><svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-5 w-5 mr-2"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M4 16v2a2 2 0 002 2h12a2 2 0 002-2v-2m-4-4l-4 4m0 0l-4-4m4 4V4"
                    />
                  </svg></a
                >
              </div>
            </div>
          </div>
        </div>
      {/each}
    </div>
    {#if loading}
      <div
        class="absolute inset-0 bg-base-100 bg-opacity-50 flex items-center justify-center"
        transition:fade
      >
        <div class="loading loading-spinner loading-lg"></div>
      </div>
    {/if}
    {#if displayedData.length === filteredData.length}
      <p class="text-center mt-4">No more items to load</p>
    {/if}
  </div>
</div>
