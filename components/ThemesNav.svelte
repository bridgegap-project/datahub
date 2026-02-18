<script>
    import { buildQuery } from "@evidence-dev/component-utilities/buildQuery";
    import { QueryLoad, Skeleton } from '@evidence-dev/core-components';

    // import ChevronCircleUp from 'svelte-icons/fa/FaChevronCircleUp.svelte';
    const qThemes = buildQuery("select * from datahubGsheets.dh_Themes", "nonssr");   
	const qComponents = buildQuery("select * from datahubGsheets.dh_Components", "nonssr2");   
	const qIndicators = buildQuery("select * from datahubGsheets.dh_Indicators", "nonssr3");

    // Mobile menu state
    let mobileMenuOpen = false;
    let openTheme = null;
    let openComponent = null;

    function toggleMobileMenu() {
        mobileMenuOpen = !mobileMenuOpen;
        if (!mobileMenuOpen) {
            openTheme = null;
            openComponent = null;
        }
    }

    function toggleTheme(themeId) {
        openTheme = openTheme === themeId ? null : themeId;
        openComponent = null; // Close any open component when switching themes
    }

    function toggleComponent(componentId) {
        openComponent = openComponent === componentId ? null : componentId;
    }
</script>
  
<nav class="relative">
    <!-- Mobile header with hamburger and logo -->
    <div class="md:hidden flex items-center justify-between px-4 bg-slate-900">
        <button 
            id="mobileToggle"
            class="p-2 text-white focus:outline-none focus:ring-2 focus:ring-white rounded"
            on:click={toggleMobileMenu}
            aria-label="Toggle navigation"
        >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                {#if mobileMenuOpen}
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                {:else}
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
                {/if}
            </svg>
        </button>
        
        <!-- Mobile logo -->
        <div class="flex items-center">
            <span class="text-white font-semibold ">Menu</span>
        </div>
        
        <!-- Spacer to center logo -->
        <div class="w-10"></div>
    </div>

    <!-- Navigation content -->
    <div class="themesWrapper {mobileMenuOpen ? 'block' : 'hidden'} md:block min-w-full">
        <QueryLoad data={qThemes} let:loaded>
            <svelte:fragment slot="skeleton" />
            
            <!-- Desktop horizontal layout -->
            <ul class="hidden md:flex gap-1">
                {#each loaded as theme}
                <div class="Theme group relative"> 
                    <button 
                        aria-haspopup="true" 
                        aria-controls="menu-{theme.id}"
                        class="outline-none focus:outline-none text-sm p-3 bg-slate-900 rounded-sm flex items-center min-w-40 hover:bg-slate-800 transition-colors"
                    > 
                        <span class="flex-1">{theme.Name}</span>
                    </button>
                    <ul 
                        id="menu-{theme.id}"
                        class="bg-slate-800 border rounded-sm transform scale-0 group-hover:scale-100 absolute origin-top min-w-80 z-10 top-full left-0"
                    >
                        <QueryLoad data={qComponents} let:loaded>
                            {#each loaded as component}
                            {#if component.Theme == theme.id}
                            <li class="Component rounded-sm relative hover:bg-slate-700"> 
                                <button 
                                    aria-haspopup="true" 
                                    aria-controls="menu-component-{component.id}"
                                    class="w-full px-3 py-1 text-left flex items-center outline-none focus:outline-none text-white hover:bg-slate-600 transition-colors"
                                >
                                    <span class="pr-1 flex-1">{component.Name}</span>
                                    <span class="text-xs">›</span>
                                </button>
                                <ul 
                                    id="menu-component-{component.id}"
                                    class="component-submenu absolute top-0 left-full bg-purple-700 border rounded-sm min-w-64 w-80 z-20"
                                >
                                    <QueryLoad data={qIndicators} let:loaded>
                                    {#each loaded as indicator}
                                    {#if indicator.Component == component.id}
                                    <li class="Indicator hover:bg-sky-900">
                                        <a class="px-3 py-1 block text-white hover:bg-sky-800 transition-colors" href="/indicator/{indicator.id}">{indicator.Name}</a>
                                    </li>
                                    {/if}            
                                    {/each}
                                    </QueryLoad>
                                </ul>
                            </li> 
                            {/if}
                            {/each}
                        </QueryLoad>
                    </ul>      
                </div>
                {/each}
            </ul>

            <!-- Mobile vertical accordion layout -->
            <div class="md:hidden bg-slate-900 w-full min-h-screen">
                {#each loaded as theme}
                <div class="Theme border-b border-slate-700 last:border-b-0">
                    <button 
                        class="w-full px-4 py-3 text-left flex items-center justify-between text-white hover:bg-slate-800 transition-colors"
                        on:click={() => toggleTheme(theme.id)}
                        aria-expanded={openTheme === theme.id}
                    >
                        <span>{theme.Name}</span>
                        <svg 
                            class="w-5 h-5 transform transition-transform {openTheme === theme.id ? 'rotate-90' : ''}"
                            fill="none" 
                            stroke="currentColor" 
                            viewBox="0 0 24 24"
                        >
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                        </svg>
                    </button>
                    
                    {#if openTheme === theme.id}
                    <div class="bg-slate-800">
                        <QueryLoad data={qComponents} let:loaded>
                            {#each loaded as component}
                            {#if component.Theme == theme.id}
                            <div class="Component border-b border-slate-700 last:border-b-0">
                                <button 
                                    class="w-full px-6 py-2 text-left flex items-center justify-between text-white hover:bg-slate-700 transition-colors"
                                    on:click={() => toggleComponent(component.id)}
                                    aria-expanded={openComponent === component.id}
                                >
                                    <span>{component.Name}</span>
                                    <svg 
                                        class="w-4 h-4 transform transition-transform {openComponent === component.id ? 'rotate-90' : ''}"
                                        fill="none" 
                                        stroke="currentColor" 
                                        viewBox="0 0 24 24"
                                    >
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                                    </svg>
                                </button>
                                
                                {#if openComponent === component.id}
                                <div class="bg-purple-700">
                                    <QueryLoad data={qIndicators} let:loaded>
                                    {#each loaded as indicator}
                                    {#if indicator.Component == component.id}
                                    <div class="Indicator">
                                        <a 
                                            class="block px-8 py-2 text-white hover:bg-sky-700 transition-colors" 
                                            href="/indicator/{indicator.id}"
                                            on:click={toggleMobileMenu}
                                        >
                                            {indicator.Name}
                                        </a>
                                    </div>
                                    {/if}            
                                    {/each}
                                    </QueryLoad>
                                </div>
                                {/if}
                            </div>
                            {/if}
                            {/each}
                        </QueryLoad>
                    </div>
                    {/if}
                </div>
                {/each}
            </div>
        </QueryLoad>
    </div>
</nav>

<style>
    /* Desktop hover effects */
    @media (min-width: 768px) {
        .group:hover .scale-0 {
            transform: scale(1);
        }
        
        /* Add hover bridge to prevent menu from closing */
        .Component {
            position: relative;
        }
        
        .Component::before {
            content: '';
            position: absolute;
            top: 0;
            right: -1px;
            width: 1px;
            height: 100%;
            background: transparent;
            z-index: 21;
        }
        
        .Component:hover .component-submenu {
            transform: translateX(0%) translateY(0px) scale(1);
        }
        
        .component-submenu {
            transform: translateX(100%) scale(0);
            transition: transform 0.15s ease-in-out;
        }
        
        /* Keep submenu open when hovering over it */
        .component-submenu:hover {
            transform: translateX(0%) translateY(0px) scale(1);
        }
    }
    
    /* Hide empty menus */
    ul:empty,
    li.Component button:has(+ :empty) {
        display: none;
    }
    
    /* Mobile menu positioning */
    @media (max-width: 767px) {
        .themesWrapper {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            z-index: 50;
            padding-top: 4rem; /* Account for header height */
        }
    }
</style>