<!-- Matomo -->
<script>
  import { browser } from '$app/environment';
  import { onMount } from 'svelte';

  onMount(() => {
    if (browser) {
      var _paq = window._paq = window._paq || [];
      /* tracker methods like "setCustomDimension" should be called before "trackPageView" */
      _paq.push(["setDocumentTitle", window.location.hostname + "/" + document.title]);
      _paq.push(["setCookieDomain", "*.corruptiondata.eu"]);
      _paq.push(["setDomains", ["*.corruptiondata.eu","*.bridgegap-project.eu"]]);
      _paq.push(['trackPageView']);
      _paq.push(['enableLinkTracking']);
      (function() {
        var u="https://lisboncouncilresearch.com/matomo/";
        _paq.push(['setTrackerUrl', u+'matomo.php']);
        _paq.push(['setSiteId', '7']);
        var d=document, g=d.createElement('script'), s=d.getElementsByTagName('script')[0];
        g.async=true; g.src=u+'matomo.js'; s.parentNode.insertBefore(g,s);
      })();
      console.log('mtm');
    }
  });
</script>

<!-- End Matomo Code -->