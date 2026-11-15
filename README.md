# youtube-with-out-shorts-option
remove your doom scrolling


import { useEffect, useRef, useState } from 'react';
import {
  ActivityIndicator,
  BackHandler,
  Platform,
  Pressable,
  StyleSheet,
  Text,
  View,
} from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { WebView } from 'react-native-webview';

type Site = 'instagram' | 'youtube' | null;

// INSTAGRAM SAFE MODE
const instagramFilter = `
(function () {
  if (window.__focusSocialInstagramInstalled) return true;
  window.__focusSocialInstagramInstalled = true;

  const style = document.createElement('style');

  style.textContent = \`
    a[href="/reels/"],
    a[href^="/explore"]:not([href^="/explore/search"]),
    a[href="/explore/search/"],
    [aria-label*="Reels" i],
    [title*="Reels" i] {
      display: none !important;
    }
  \`;

  (document.head || document.documentElement).appendChild(style);

 

  true;
})();
`;

// YOUTUBE FOCUS MODE
const youtubeFilter = `
(function () {
  if (window.__focusSocialYouTubeInstalled) return true;
  window.__focusSocialYouTubeInstalled = true;

  const style = document.createElement('style');

  style.textContent = \`
    ytd-reel-shelf-renderer,
    ytd-reel-item-renderer,
    ytm-reel-shelf-renderer,
    ytm-reel-item-renderer,
    a[href^="/shorts"],
    a[href*="/shorts/"],
    [aria-label*="Shorts" i] {
      display: none !important;
    }
  \`;

  (document.head || document.documentElement).appendChild(style);

  true;
})();
`;

export default function HomeScreen() {
  const webRef = useRef<WebView>(null);

  const [site, setSite] = useState<Site>(null);
  const [loading, setLoading] = useState(false);
  const [canGoBack, setCanGoBack] = useState(false);
  

  const goHome = () => {
    setSite(null);
    setLoading(false);
    setCanGoBack(false);
  };

  // Android back button
  useEffect(() => {
    if (Platform.OS !== 'android') return;

    const sub = BackHandler.addEventListener(
      'hardwareBackPress',
      () => {
        if (site && canGoBack) {
          webRef.current?.goBack();
          return true;
        }

        if (site) {
          goHome();
          return true;
        }

        return false;
      }
    );

    return () => sub.remove();
  }, [site, canGoBack]);

  const openSite = (selected: Site) => {
    setSite(selected);
    setLoading(true);
  };

  // Instagram Search
  const openInstagramSearch = () => {
    webRef.current?.injectJavaScript(`
      window.location.href =
        'https://www.instagram.com/explore/search/';
      true;
    `);
  };

  // HOME DASHBOARD
  if (!site) {
    return (
      <SafeAreaView style={styles.safe}>
        <View style={styles.home}>

          <View style={styles.brand}>
            <View style={styles.logo}>
              <Text style={styles.logoText}>F</Text>
            </View>

            <Text style={styles.title}>FocusSocial</Text>

            <Text style={styles.subtitle}>
              Your feed. Your focus.
            </Text>
          </View>

          <View style={styles.cards}>

            <Pressable
              style={styles.card}
              onPress={() => openSite('instagram')}
            >
              <View style={styles.instagramIcon}>
                <Text style={styles.iconText}>◎</Text>
              </View>

              <View style={styles.cardContent}>
                <Text style={styles.cardTitle}>Instagram</Text>
                <Text style={styles.cardSubtitle}>
                  Safe Mode · No Reels feed
                </Text>
              </View>

              <Text style={styles.arrow}>›</Text>
            </Pressable>

            <Pressable
              style={styles.card}
              onPress={() => openSite('youtube')}
            >
              <View style={styles.youtubeIcon}>
                <Text style={styles.playIcon}>▶</Text>
              </View>

              <View style={styles.cardContent}>
                <Text style={styles.cardTitle}>YouTube</Text>
                <Text style={styles.cardSubtitle}>
                  Focus Mode · No Shorts feed
                </Text>
              </View>

              <Text style={styles.arrow}>›</Text>
            </Pressable>

          </View>

          <View style={styles.footer}>
            <Text style={styles.footerTitle}>
              Designed for intentional scrolling.
            </Text>

            <Text style={styles.footerText}>
              Less distraction. More control.
            </Text>
          </View>

        </View>
      </SafeAreaView>
    );
  }

  // WEBSITE URLS
  const url =
    site === 'instagram'
      ? 'https://www.instagram.com/'
      : 'https://www.youtube.com/feed/subscriptions';

  // WEBSITE VIEW
  return (
    <SafeAreaView style={styles.safe}>

      <View style={styles.topBar}>
  <Pressable onPress={goHome}>
    <Text style={styles.backText}>‹ Back</Text>
  </Pressable>

  <Text style={styles.topTitle}>
    {site === 'instagram' ? 'Instagram Safe Mode' : 'YouTube Focus Mode'}
  </Text>

  <View style={{ width: 50 }} />
</View>

{site === 'instagram' && (
  <Pressable
    style={styles.searchButton}
    onPress={openInstagramSearch}
  >
    <Text style={styles.searchText}>🔍</Text>
  </Pressable>
)}

      {loading && (
        <View style={styles.loading}>
          <ActivityIndicator color="#9385FF" />
        </View>
      )}


      <WebView
        ref={webRef}
        source={{ uri: url }}
        style={styles.webview}

        javaScriptEnabled
        domStorageEnabled
        sharedCookiesEnabled
        thirdPartyCookiesEnabled

        allowsBackForwardNavigationGestures
        allowsInlineMediaPlayback={true}
        mediaPlaybackRequiresUserAction={true}

        injectedJavaScriptBeforeContentLoaded={
          site === 'instagram'
            ? instagramFilter
            : youtubeFilter
        }

        injectedJavaScript={
          site === 'instagram'
            ? instagramFilter
            : youtubeFilter
        }

        onLoadStart={() => setLoading(true)}
        onLoadEnd={() => setLoading(false)}

        onNavigationStateChange={(nav) => {
          setCanGoBack(nav.canGoBack);
         
        }}

        originWhitelist={['https://*']}
        setSupportMultipleWindows={false}
      />

    </SafeAreaView>
  );
}

// DESIGN
const styles = StyleSheet.create({
  safe: {
    flex: 1,
    backgroundColor: '#090B10',
  },

  home: {
    flex: 1,
    paddingHorizontal: 24,
    paddingTop: 45,
    paddingBottom: 25,
    justifyContent: 'space-between',
  },

  brand: {
    alignItems: 'center',
    marginTop: 35,
  },

  logo: {
    width: 65,
    height: 65,
    borderRadius: 22,
    backgroundColor: '#8878FF',
    alignItems: 'center',
    justifyContent: 'center',
    marginBottom: 18,
  },

  logoText: {
    color: '#FFFFFF',
    fontSize: 36,
    fontWeight: '800',
  },

  title: {
    color: '#F5F6FA',
    fontSize: 32,
    fontWeight: '800',
  },

  subtitle: {
    color: '#9AA5B5',
    fontSize: 15,
    marginTop: 8,
  },

  cards: {
    gap: 16,
  },

  card: {
    backgroundColor: '#151B26',
    borderRadius: 22,
    padding: 18,
    flexDirection: 'row',
    alignItems: 'center',
    borderWidth: 1,
    borderColor: '#273142',
  },

  instagramIcon: {
    width: 52,
    height: 52,
    borderRadius: 17,
    backgroundColor: '#322342',
    alignItems: 'center',
    justifyContent: 'center',
    marginRight: 15,
  },

  youtubeIcon: {
    width: 52,
    height: 52,
    borderRadius: 17,
    backgroundColor: '#3A2025',
    alignItems: 'center',
    justifyContent: 'center',
    marginRight: 15,
  },

  iconText: {
    color: '#F3D6FF',
    fontSize: 32,
  },

  playIcon: {
    color: '#FF777F',
    fontSize: 23,
  },

  cardContent: {
    flex: 1,
  },

  cardTitle: {
    color: '#F5F6FA',
    fontSize: 18,
    fontWeight: '700',
  },

  cardSubtitle: {
    color: '#9AA5B5',
    fontSize: 12,
    marginTop: 5,
  },

  arrow: {
    color: '#9AA5B5',
    fontSize: 30,
  },

  footer: {
    backgroundColor: '#111722',
    padding: 18,
    borderRadius: 18,
  },

  footerTitle: {
    color: '#F5F6FA',
    fontSize: 14,
    fontWeight: '700',
  },

  footerText: {
    color: '#9AA5B5',
    fontSize: 12,
    marginTop: 6,
  },

  topBar: {
    minHeight: 54,
    backgroundColor: '#090B10',
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    paddingHorizontal: 12,
  },

  backText: {
    color: '#F5F6FA',
    fontSize: 14,
    fontWeight: '600',
  },

  topTitle: {
    color: '#F5F6FA',
    fontSize: 13,
    fontWeight: '600',
  },

  searchText: {
    color: '#9385FF',
    fontSize: 14,
    fontWeight: '700',
  },
  searchButton: {
  width: 42,
  height: 42,
  marginHorizontal: 12,
  alignItems: 'center',
  justifyContent: 'center',
},

  reload: {
    color: '#F5F6FA',
    fontSize: 32,
    fontWeight: '400',
  },

  webview: {
    flex: 1,
  },

  loading: {
    position: 'absolute',
    zIndex: 2,
    top: 65,
    alignSelf: 'center',
    backgroundColor: '#111722',
    padding: 10,
    borderRadius: 20,
  },
});
