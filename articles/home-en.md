---
layout: default
title: Home
lang: en
permalink: /articles/home-en.html
---

<div class="kb-header-section">
  <div class="kb-header-row">
    <h1 class="kb-main-title">Articles</h1>
  </div>

  <div class="kb-filters-row">
    <button data-topic-filter="all" class="kb-chip kb-chip-active">All</button>
    {% for topic in site.data.topics.topics %}
    <button data-topic-filter="{{ topic.id }}" class="kb-chip">{{ topic.name }}</button>
    {% endfor %}
    <a href="/about/" class="kb-chip kb-chip-link">About Me</a>
  </div>
</div>

<style>
.kb-header-section {
  margin-bottom: 2rem;
}

.kb-header-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1.25rem;
}

.kb-main-title {
  font-family: 'Courier Prime', monospace;
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--kb-text-primary);
  margin: 0;
}

.kb-filters-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.kb-chip {
  font-family: 'Courier Prime', monospace;
  font-size: 0.75rem;
  padding: 0.35rem 0.75rem;
  background: transparent;
  border: 1px solid var(--kb-border);
  border-radius: 999px;
  color: var(--kb-text-secondary);
  cursor: pointer;
  transition: all 0.15s ease;
  text-decoration: none;
}

.kb-chip:hover {
  border-color: var(--kb-accent);
  color: var(--kb-accent);
}

.kb-chip.kb-chip-active {
  background: var(--kb-accent);
  border-color: var(--kb-accent);
  color: var(--kb-bg);
}

.kb-chip-link {
  text-decoration: none;
}

@media (max-width: 640px) {
  .kb-header-row {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.75rem;
  }

  .kb-main-title {
    font-size: 1.25rem;
  }

  .kb-filters-row {
    gap: 0.35rem;
  }

  .kb-chip {
    font-size: 0.7rem;
    padding: 0.3rem 0.6rem;
  }
}
</style>

{% assign topics = site.data.topics.topics %}

<div id="topic-sections" class="kb-articles-container">
  {% for topic in topics %}
    {% assign posts_in_topic = site.posts | where: "topic", topic.id | where: "lang", "en" %}
    {% if posts_in_topic.size > 0 %}
    <section class="topic-section" data-topic="{{ topic.id }}">
      <div class="kb-section-header">
        <h2 class="kb-section-title">
          {{ topic.name }}
        </h2>
        <div class="kb-section-divider"></div>
      </div>
      
      <ul class="kb-post-list">
        {% for post in posts_in_topic %}
        <li class="kb-post-item" data-lang="{{ post.lang | default: 'en' }}">
          <div class="kb-post-card">
            {% if post.image %}
            <div class="kb-post-thumbnail">
              <img src="{{ post.image | relative_url }}" alt="{{ post.title }}" loading="lazy">
            </div>
            {% endif %}
            <div class="kb-post-content">
              <div class="kb-post-meta">
                <span class="kb-post-topic">{{ topic.name }}</span>
                {% if post.lang %}
                <span class="kb-post-lang">{{ post.lang | upcase }}</span>
                {% endif %}
                {% if post.date %}
                <span class="kb-post-date">{{ post.date | date: "%B %d, %Y" }}</span>
                {% endif %}
              </div>
              
              <h3 class="kb-post-title">
                <a href="{{ post.url | relative_url }}" class="kb-post-link">
                  {{ post.title }}
                </a>
              </h3>
              
              {% if post.excerpt %}
              <p class="kb-post-excerpt">
                {{ post.excerpt | strip_html | strip_newlines | truncatewords: 30 }}
              </p>
              {% endif %}
              
              {% if post.tags %}
              <div class="kb-post-tags">
                {% for tag in post.tags %}
                <span class="kb-tag">{{ tag }}</span>
                {% endfor %}
              </div>
              {% endif %}
            </div>
          </div>
        </li>
        {% endfor %}
      </ul>
    </section>
    {% endif %}
  {% endfor %}
</div>

<div id="no-results" class="kb-no-results hidden">
  <div class="kb-no-results-icon">∅</div>
  <p>No articles found matching the selected filters.</p>
</div>

{% raw %}
<script>
  document.addEventListener('DOMContentLoaded', function () {
    const topicButtons = document.querySelectorAll('[data-topic-filter]');
    const sections = document.querySelectorAll('.topic-section');
    const noResults = document.getElementById('no-results');
    
    let currentTopic = 'all';

    function updateDisplay() {
      let visibleCount = 0;
      
      sections.forEach(sec => {
        const topic = sec.dataset.topic;
        const postsInSection = sec.querySelectorAll('.kb-post-item');
        let visibleInSection = 0;
        
        postsInSection.forEach(post => {
          const topicMatch = currentTopic === 'all' || currentTopic === topic;
          
          if (topicMatch) {
            post.classList.remove('hidden');
            visibleInSection++;
            visibleCount++;
          } else {
            post.classList.add('hidden');
          }
        });
        
        const sectionMatch = currentTopic === 'all' || currentTopic === topic;
        if (visibleInSection > 0 && sectionMatch) {
          sec.classList.remove('hidden');
        } else {
          sec.classList.add('hidden');
        }
      });
      
      if (visibleCount === 0) {
        noResults.classList.remove('hidden');
      } else {
        noResults.classList.add('hidden');
      }
    }

    function setActiveTopic(filter) {
      currentTopic = filter;
      topicButtons.forEach(btn => {
        if (btn.dataset.topicFilter === filter) {
          btn.classList.add('kb-chip-active');
        } else {
          btn.classList.remove('kb-chip-active');
        }
      });
      updateDisplay();
    }
    
    topicButtons.forEach(btn => {
      btn.addEventListener('click', () => setActiveTopic(btn.dataset.topicFilter));
    });

    // Check URL parameters for topic filter (for terminal navigation)
    const urlParams = new URLSearchParams(window.location.search);
    const topicParam = urlParams.get('topic');
    if (topicParam) {
      // Find a matching button for the URL parameter
      const normalizedParam = topicParam.toLowerCase();
      let found = false;
      topicButtons.forEach(btn => {
        if (btn.dataset.topicFilter === normalizedParam) {
          found = true;
        }
      });
      if (found) {
        setActiveTopic(normalizedParam);
      }
    }

    // Initialize
    updateDisplay();
  });
</script>
{% endraw %}
