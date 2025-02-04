# Technical Documentation for WikiTok

This document provides a detailed explanation of the WikiTok project, including the main components, hooks, and their purposes, along with relevant code snippets.

## Table of Contents

- [Overview](#overview)
- [Main Components](#main-components)
  - [App Component](#app-component)
  - [WikiCard Component](#wikicard-component)
- [Custom Hooks](#custom-hooks)
  - [useWikiArticles Hook](#usewikiarticles-hook)
- [Code Snippets](#code-snippets)

## Overview

WikiTok is a TikTok-style interface for exploring Wikipedia articles. It is built using React, TypeScript, and Vite, and leverages Tailwind CSS for styling. The application fetches random Wikipedia articles and displays them in an infinite scrolling interface.

## Main Components

### App Component

The `App` component is the main component of the application. It handles the fetching of articles and manages the state of the application.

#### Code Snippet

```tsx
import { useEffect, useRef, useCallback, useState } from 'react'
import { WikiCard } from './components/WikiCard'
import { useWikiArticles } from './hooks/useWikiArticles'
import { Loader2 } from 'lucide-react'

function App() {
  const [showAbout, setShowAbout] = useState(false)
  const { articles, loading, fetchArticles } = useWikiArticles()
  const observerTarget = useRef(null)

  const handleObserver = useCallback(
    (entries: IntersectionObserverEntry[]) => {
      const [target] = entries
      if (target.isIntersecting && !loading) {
        fetchArticles()
      }
    },
    [loading, fetchArticles]
  )

  useEffect(() => {
    const observer = new IntersectionObserver(handleObserver, {
      threshold: 0.5,
    })

    if (observerTarget.current) {
      observer.observe(observerTarget.current)
    }

    return () => observer.disconnect()
  }, [handleObserver])

  useEffect(() => {
    fetchArticles()
  }, [])

  return (
    <div className="h-screen w-full bg-black text-white overflow-y-scroll snap-y snap-mandatory">
      <div className="fixed top-4 left-4 z-50">
        <button
          onClick={() => window.location.reload()}
          className="text-2xl font-bold text-white drop-shadow-lg hover:opacity-80 transition-opacity"
        >
          WikiTok
        </button>
      </div>

      <div className="fixed top-4 right-4 z-50">
        <button
          onClick={() => setShowAbout(!showAbout)}
          className="text-sm text-white/70 hover:text-white transition-colors"
        >
          About
        </button>
      </div>

      {showAbout && (
        <div className="fixed inset-0 bg-black/80 backdrop-blur-sm z-50 flex items-center justify-center p-4">
          <div className="bg-gray-900 p-6 rounded-lg max-w-md relative">
            <button
              onClick={() => setShowAbout(false)}
              className="absolute top-2 right-2 text-white/70 hover:text-white"
            >
              ✕
            </button>
            <h2 className="text-xl font-bold mb-4">About WikiTok</h2>
            <p className="mb-4">A TikTok-style interface for exploring Wikipedia articles.</p>
            <p className="text-white/70">
              Made with ❤️ by{' '}
              <a
                href="https://x.com/Aizkmusic"
                target="_blank"
                rel="noopener noreferrer"
                className="text-white hover:underline"
              >
                @Aizkmusic
              </a>
            </p>
          </div>
        </div>
      )}

      {articles.map((article) => (
        <WikiCard key={article.pageid} article={article} />
      ))}
      <div ref={observerTarget} className="h-10" />
      {loading && (
        <div className="h-screen w-full flex items-center justify-center gap-2">
          <Loader2 className="h-6 w-6 animate-spin" />
          <span>Loading...</span>
        </div>
      )}
    </div>
  )
}

export default App
```

### WikiCard Component

The `WikiCard` component is responsible for displaying individual Wikipedia articles. It includes the article title, a brief extract, and a thumbnail image if available.

#### Code Snippet

```tsx
import { Share2 } from 'lucide-react';
import { useState, useEffect } from 'react';

interface WikiArticle {
    title: string;
    pageid: number;
    thumbnail?: {
        source: string;
        width: number;
        height: number;
    };
}

interface WikiCardProps {
    article: WikiArticle;
}

export function WikiCard({ article }: WikiCardProps) {
    const [imageLoaded, setImageLoaded] = useState(false);
    const [articleContent, setArticleContent] = useState<string | null>(null);

    useEffect(() => {
        const fetchArticleContent = async () => {
            try {
                const response = await fetch(
                    `https://en.wikipedia.org/w/api.php?` +
                    `action=query&format=json&origin=*&prop=extracts&` +
                    `pageids=${article.pageid}&explaintext=1&exintro=1&` +
                    `exsentences=5`  // Limit to 5 sentences
                );
                const data = await response.json();
                const content = data.query.pages[article.pageid].extract;
                if (content) {
                    setArticleContent(content);
                }
            } catch (error) {
                console.error('Error fetching article content:', error);
            }
        };

        fetchArticleContent();
    }, [article.pageid]);

    // Add debugging log
    console.log('Article data:', {
        title: article.title,
        pageid: article.pageid
    });

    const handleShare = async () => {
        if (navigator.share) {
            try {
                await navigator.share({
                    title: article.title,
                    text: articleContent || '',
                    url: `https://en.wikipedia.org/?curid=${article.pageid}`
                });
            } catch (error) {
                console.error('Error sharing:', error);
            }
        } else {
            // Fallback: Copy to clipboard
            const url = `https://en.wikipedia.org/?curid=${article.pageid}`;
            await navigator.clipboard.writeText(url);
            alert('Link copied to clipboard!');
        }
    };

    return (
        <div className="h-screen w-full flex items-center justify-center snap-start relative">
            <div className="h-full w-full relative">
                {article.thumbnail ? (
                    <div className="absolute inset-0">
                        <img
                            src={article.thumbnail.source}
                            alt={article.title}
                            className={`w-full h-full object-cover transition-opacity duration-300 ${imageLoaded ? 'opacity-100' : 'opacity-0'
                                }`}
                            onLoad={() => setImageLoaded(true)}
                            onError={(e) => {
                                console.error('Image failed to load:', e);
                                setImageLoaded(true); // Show content even if image fails
                            }}
                        />
                        {!imageLoaded && (
                            <div className="absolute inset-0 bg-gray-900 animate-pulse" />
                        )}
                        <div className="absolute inset-0 bg-gradient-to-b from-transparent to-black/70" />
                    </div>
                ) : (
                    <div className="absolute inset-0 bg-gray-900" />
                )}
                {/* Content container with z-index to ensure it's above the image */}
                <div className="absolute bottom-[10vh] left-0 right-0 p-6 text-white z-10">
                    <div className="flex justify-between items-start mb-3">
                        <a
                            href={`https://en.wikipedia.org/?curid=${article.pageid}`}
                            target="_blank"
                            rel="noopener noreferrer"
                            className="hover:text-gray-200 transition-colors"
                        >
                            <h2 className="text-2xl font-bold drop-shadow-lg">{article.title}</h2>
                        </a>
                        <button
                            onClick={handleShare}
                            className="p-2 rounded-full bg-white/10 backdrop-blur-sm hover:bg-white/20 transition-colors"
                            aria-label="Share article"
                        >
                            <Share2 className="w-5 h-5" />
                        </button>
                    </div>
                    {articleContent ? (
                        <p className="text-gray-100 mb-4 drop-shadow-lg">{articleContent}</p>
                    ) : (
                        <p className="text-gray-100 mb-4 drop-shadow-lg italic">Loading description...</p>
                    )}
                    <a
                        href={`https://en.wikipedia.org/?curid=${article.pageid}`}
                        target="_blank"
                        rel="noopener noreferrer"
                        className="inline-block text-white hover:text-gray-200 drop-shadow-lg"
                    >
                        Read more →
                    </a>
                </div>
            </div>
        </div>
    );
} 
```

## Custom Hooks

### useWikiArticles Hook

The `useWikiArticles` hook is a custom hook that handles the fetching of random Wikipedia articles. It manages the state of the articles and the loading state.

#### Code Snippet

```tsx
import { useState } from "react";

interface WikiArticle {
  title: string;
  extract: string;
  pageid: number;
  thumbnail?: {
    source: string;
    width: number;
    height: number;
  };
}

const preloadImage = (src: string): Promise<void> => {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.src = src;
    img.onload = () => resolve();
    img.onerror = reject;
  });
};

export function useWikiArticles() {
  const [articles, setArticles] = useState<WikiArticle[]>([]);
  const [loading, setLoading] = useState(false);

  const fetchArticles = async () => {
    setLoading(true);
    try {
      const response = await fetch(
        "https://en.wikipedia.org/w/api.php?" +
          new URLSearchParams({
            action: "query",
            format: "json",
            generator: "random",
            grnnamespace: "0",
            prop: "extracts|pageimages",
            grnlimit: "40",
            exintro: "1",
            exchars: "1000",
            exlimit: "max",
            explaintext: "1",
            piprop: "thumbnail",
            pithumbsize: "400",
            origin: "*",
          })
      );

      const data = await response.json();
      const newArticles = Object.values(data.query.pages)
        .map((page: any) => ({
          title: page.title,
          extract: page.extract,
          pageid: page.pageid,
          thumbnail: page.thumbnail,
        }))
        .filter((article) => article.thumbnail);

      await Promise.allSettled(
        newArticles
          .filter((article) => article.thumbnail)
          .map((article) => preloadImage(article.thumbnail!.source))
      );

      setArticles((prev) => [...prev, ...newArticles]);
    } catch (error) {
      console.error("Error fetching articles:", error);
    }
    setLoading(false);
  };

  return { articles, loading, fetchArticles };
}
```

## Code Snippets

This section includes code snippets for each functionality and explains their purposes.

### Infinite Scrolling

The infinite scrolling functionality is implemented using the `IntersectionObserver` API. The `App` component sets up an observer that triggers the `fetchArticles` function when the user scrolls to the bottom of the page.

#### Code Snippet

```tsx
const handleObserver = useCallback(
  (entries: IntersectionObserverEntry[]) => {
    const [target] = entries
    if (target.isIntersecting && !loading) {
      fetchArticles()
    }
  },
  [loading, fetchArticles]
)

useEffect(() => {
  const observer = new IntersectionObserver(handleObserver, {
    threshold: 0.5,
  })

  if (observerTarget.current) {
    observer.observe(observerTarget.current)
  }

  return () => observer.disconnect()
}, [handleObserver])
```

### Article Sharing

The article sharing functionality is implemented using the Web Share API. If the Web Share API is not available, the URL is copied to the clipboard as a fallback.

#### Code Snippet

```tsx
const handleShare = async () => {
  if (navigator.share) {
    try {
      await navigator.share({
        title: article.title,
        text: articleContent || '',
        url: `https://en.wikipedia.org/?curid=${article.pageid}`
      });
    } catch (error) {
      console.error('Error sharing:', error);
    }
  } else {
    // Fallback: Copy to clipboard
    const url = `https://en.wikipedia.org/?curid=${article.pageid}`;
    await navigator.clipboard.writeText(url);
    alert('Link copied to clipboard!');
  }
};
```

### Fetching Article Content

The `WikiCard` component fetches the content of the article using the Wikipedia API. The content is limited to 5 sentences for brevity.

#### Code Snippet

```tsx
useEffect(() => {
  const fetchArticleContent = async () => {
    try {
      const response = await fetch(
        `https://en.wikipedia.org/w/api.php?` +
        `action=query&format=json&origin=*&prop=extracts&` +
        `pageids=${article.pageid}&explaintext=1&exintro=1&` +
        `exsentences=5`  // Limit to 5 sentences
      );
      const data = await response.json();
      const content = data.query.pages[article.pageid].extract;
      if (content) {
        setArticleContent(content);
      }
    } catch (error) {
      console.error('Error fetching article content:', error);
    }
  };

  fetchArticleContent();
}, [article.pageid]);
```

### Preloading Images

The `useWikiArticles` hook preloads the images of the articles to ensure a smooth user experience.

#### Code Snippet

```tsx
const preloadImage = (src: string): Promise<void> => {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.src = src;
    img.onload = () => resolve();
    img.onerror = reject;
  });
};

await Promise.allSettled(
  newArticles
    .filter((article) => article.thumbnail)
    .map((article) => preloadImage(article.thumbnail!.source))
);
```

This concludes the technical documentation for the WikiTok project. For any further questions or clarifications, please refer to the source code or contact the project maintainer.
