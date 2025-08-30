# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a front-end test project containing a comprehensive hero video implementation demo. The project consists of a single, well-documented HTML file that demonstrates modern video hero section techniques with advanced CSS and JavaScript.

## Architecture

- **Single-file architecture**: All code (HTML, CSS, JavaScript) is contained in `hero_video_implementation.html`
- **Modular CSS**: Styles are organized by component with clear section headers and extensive commenting
- **Class-based JavaScript**: Uses ES6 class `HeroVideoManager` for video state management
- **Layered approach**: Content is structured with z-index layers (background → video → overlay → content → loading indicator)

## Key Components

### CSS Structure
- Reset and base styles
- Hero section with responsive design
- Video placeholder with animated gradients
- Overlay system for text readability
- Responsive breakpoints (1024px, 768px, 480px)
- Accessibility features (prefers-reduced-motion support)

### JavaScript Features
- `HeroVideoManager` class handles all video functionality
- Performance optimizations (connection detection, intersection observer)
- Accessibility support (reduced motion preferences)
- Error handling and fallback systems
- Mobile-specific optimizations

## Development Commands

Since this is a static HTML project, development is straightforward:
- Open `hero_video_implementation.html` directly in browser
- Use browser dev tools for debugging
- Check browser console for detailed logging output

## Technical Details

### Video Implementation
- Uses MDN's rabbit320.webm as demo video
- Implements proper fallback chain: video → placeholder → static image
- Supports autoplay with graceful degradation
- Object-fit: cover for responsive video scaling

### Performance Features
- Network-aware loading strategies
- Intersection Observer for visibility-based play/pause
- Mobile/desktop adaptive behavior
- Preload strategies based on connection speed

### Browser Compatibility
- Modern browsers (ES6+ support required)
- Graceful fallbacks for older browsers
- Cross-platform video format support (WebM primary)

## Code Conventions

- Extensive commenting throughout all sections
- French language used in comments and UI text
- Console logging for debugging and education
- Clear separation of concerns between HTML structure, CSS presentation, and JavaScript behavior