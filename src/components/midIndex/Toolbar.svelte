<script>
    export let showModeBtn = true

    import { onMount } from 'svelte'
    import { createEventDispatcher } from 'svelte'
    import { saveViewScope } from '../utils/storage.js'
    import { activeTab, rssViewMode, viewScope, starViewMode, isApiLoading } from '../utils/store.js'
    import { shortToast, warnToast } from '../utils/helper.js'
    import FeedCard from '../global/FeedCard.svelte'
    import { apiReq } from '../utils/req.js';

    const Mousetrap = require('mousetrap')
    const dispatch = createEventDispatcher()

    let searchKeyword
    let searchRsp

    onMount(() => {
        // keyboard shortcut
        Mousetrap.bind('r', function() {
            handleRefreshAction()
            shortToast("Refresh")
            return false
        });
    })

    function handleToggleViewMode() {
        // change status after network
        if ($activeTab === "rss") {
            dispatch('refresh-list-view', {page: 1, mode: ($rssViewMode === 'feed') ? 'entry' : 'feed'})
        } else if ($activeTab === "star") {
            dispatch('refresh-list-view', {page: 1, mode: ($starViewMode === 'feed') ? 'entry' : 'feed'})
        }
    }
    function handleToggleViewScope() {
        // change status instant
        viewScope.set(($viewScope === 'all') ? 'unread' : 'all')
        saveViewScope($viewScope)
        dispatch('refresh-list-view', {page: 1})
    }
    function handleRefreshAction() {
        dispatch('refresh-list-view', {page: 1})
    }
    function handleSearch(event) {
        if (event.keyCode === 13 && searchKeyword) {
            event.preventDefault()

            isApiLoading.set(true)
            searchRsp = undefined
            apiReq('/api/search/feed', {keyword: searchKeyword}).then( rsp => {
                if (rsp.code === 0) {
                    searchRsp = rsp
                    showSearchhWindow()
                } else if (rsp.code === 106) {
                    warnToast("Keyword error!")
                } else if (rsp.code === 100) {
                    warnToast("No data!")
                } 
            }).catch(err => {
                warnToast(err)
            }).finally(() => {
                isApiLoading.set(false)
            });
        }
    }
    function showSearchhWindow() {
        const instanse = M.Modal.init(document.querySelector('#omr-modal-search-feed'), {
            inDuration: 0,
            outDuration: 0,
            opacity: 0.5,
            endingTop: document.querySelector('#omr-top-toolbar').offsetHeight + 'px'
        });
        instanse.open()
    }
</script>

<style>
    #omr-top-toolbar {
        width: 100%;
        max-width: 400px;
        height: 60px;
        display: flex;
        padding-top: 16px;
        padding-bottom: 10px;
    }
    .toolbar-group {
        display: flex;
        align-items: center;
        justify-content: flex-end;
    }
    .omr-search-form {
        margin-left: 15px;
        background: #E6E6E6;
        border-radius: 3px;
    }
    input[id=omr-search-input]:active, input[id=omr-search-input]:focus {
        background-color: #E6E6E6;
    }
    .omr-search-form.input-field {
        margin: 0;
        height: 100%;
    }
    .nav-wrapper {
        width: 100%;
        flex-grow: 1;
        margin-right: 10px;
    }
    .omr-full-search {
        padding-right: 0;
        margin-right: 0 !important;
    }
    #omr-search-input {
        margin: 0;
        line-height: 34px;
        border-radius: 3px;
        font-size: .95rem;
        vertical-align: middle;
    }

    .toolbar-icon {
        width: 54px;
        height: 34px;
        color: #101010;
    }
    .toolbar-icon i {
        margin: 5px 15px;
    }
    .search-icon {
        margin: 5px 0;
    }
    #omr-modal-search-feed {
        width: 76%;
        padding: 28px;
        left: 70px;
        max-height: calc(100% - 120px);
    }
</style>

<div id="omr-top-toolbar" class="drag">
    {#if $activeTab === 'apps'}
        <div class="nav-wrapper no-drag omr-full-search">
            <div class="input-field omr-search-form">
                <input id="omr-search-input" type="search" class="" placeholder="Search" required 
                    bind:value={searchKeyword} on:keyup={handleSearch} />
                <label class="label-icon search-icon" for="omr-search-input">
                    <i class="material-icons">search</i></label>
            </div>
        </div>
    {:else}
        <div class="nav-wrapper no-drag">
            <div class="input-field omr-search-form">
                <input id="omr-search-input" type="search" class="" placeholder="Search" required
                    bind:value={searchKeyword} on:keyup={handleSearch} />
                <label class="label-icon search-icon" for="omr-search-input">
                    <i class="material-icons">search</i></label>
            </div>
        </div>

        <div class="toolbar-group no-drag">
            {#if $activeTab === 'rss'}
                <div title="Toggle unread / all" class="toolbar-icon" id="omr-toolbar-scope" on:click={handleToggleViewScope}>
                    <i class="material-icons">{ $viewScope === 'all' ? 'donut_large' : 'fiber_manual_record' }</i>
                </div>

                {#if showModeBtn}
                <div title="Toggle feed / entry view" class="toolbar-icon" id="omr-toolbar-mode" on:click={handleToggleViewMode}>
                    <i class="material-icons">{$rssViewMode === 'feed' ? 'view_module' : 'view_list'} </i>
                </div>
                {/if}
            {:else if $activeTab === 'star' && showModeBtn}
                <div title="Toggle feed / entry view" class="toolbar-icon" id="omr-toolbar-mode" on:click={handleToggleViewMode}>
                    <i class="material-icons">{$starViewMode === 'feed' ? 'view_module' : 'view_list'} </i>
                </div>
            {/if}

            <div title="Refresh" class="toolbar-icon" id="omr-toolbar-update"  on:click={handleRefreshAction}>
                <i class="material-icons">update</i>
            </div>

        </div>
    {/if}
</div>

<div id="omr-modal-search-feed" class="modal">
    <div class="modal-title"><i class="material-icons">search</i> Search "{searchKeyword}"</div>

    {#if searchRsp}
    <div class="row">
        {#each searchRsp.data as feed}
        <div class="col s6">
            <FeedCard feedInfo={feed} />
        </div>
        {/each}
    </div>
    {/if}
</div>
