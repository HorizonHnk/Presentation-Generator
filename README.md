# BET Slides AI - Presentation Generator

[![Live Demo](https://img.shields.io/badge/Live%20Demo-Visit%20Site-blue?style=for-the-badge)](https://power-point-presentation.netlify.app/)
[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?style=for-the-badge&logo=github)](https://github.com/HorizonHnk/Presentation-Generator.git)

An advanced AI-powered presentation generator built with React, Vite, and Google's Gemini API. This application creates professional PowerPoint presentations with AI-generated content, high-quality stock images from Pixabay, speaker notes, and text-to-speech coaching.

🔗 **Live Demo**: [https://power-point-presentation.netlify.app/](https://power-point-presentation.netlify.app/)

## Features

- **AI-Powered Content Generation**: Uses Gemini 2.5 Flash to create professional presentation content
- **Intelligent Design**: Follows the 6x6 rule and presentation best practices
- **Speaker Coach**: AI-generated speaker notes with text-to-speech narration
- **High-Quality Stock Images**: Automatic image integration using Pixabay API with CORS support
- **Cloud Sync**: Firebase integration for saving and managing presentations (optimized for 1MB Firestore limit)
- **Multiple Export Options**: Export to PPTX with embedded images or print as PDF
- **Responsive Design**: Works on desktop and mobile devices
- **Reference Files**: Upload PDFs, text files, or images as context
- **On-the-Fly Image Conversion**: Images converted to base64 during export to optimize storage

## Prerequisites

Before you begin, ensure you have the following installed:
- Node.js (v18 or higher)
- npm or yarn package manager

## Installation

1. Navigate to the project directory:
```bash
cd vite-presentation-app
```

2. Install dependencies:
```bash
npm install
```

## Configuration

### Environment Variables

Create a `.env` file in the root directory with the following variables:

```env
# Google Gemini API Configuration
VITE_GEMINI_API_KEY=your_gemini_api_key_here
VITE_GEMINI_MODEL=gemini-2.5-flash-preview-09-2025

# Pixabay API Configuration
VITE_PIXABAY_API_KEY=your_pixabay_api_key_here
```

**⚠️ SECURITY WARNING**: Never commit the `.env` file to Git. It contains sensitive API keys and is already included in `.gitignore`.

### Getting API Keys

#### Google Gemini API Key

1. Visit [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Create a new API key
3. Add it to your `.env` file as `VITE_GEMINI_API_KEY`

#### Pixabay API Key

1. Visit [Pixabay API](https://pixabay.com/api/docs/)
2. Sign up for a free account
3. Get your API key from the dashboard
4. Add it to your `.env` file as `VITE_PIXABAY_API_KEY`
5. Free tier: 100 requests per minute

### Firebase Configuration (Optional)

The app comes pre-configured with Firebase for cloud storage. If you want to use your own Firebase project:

1. Create a new project at [Firebase Console](https://console.firebase.google.com/)
2. Enable Authentication (Google Sign-In)
3. Enable Firestore Database
4. Update the Firebase configuration in `src/App.jsx`:

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID",
  measurementId: "YOUR_MEASUREMENT_ID"
};
```

## Development

Start the development server:

```bash
npm run dev
```

The application will open at `http://localhost:5173` (or another port if 5173 is busy).

## Building for Production

Build the application for production:

```bash
npm run build
```

Preview the production build:

```bash
npm run preview
```

## Usage

1. **Create a Presentation**:
   - Enter your topic
   - Optionally attach reference files (PDFs, text, images)
   - Select slide count and style
   - Click "Generate Deck"

2. **Review and Edit**:
   - Navigate through slides using the sidebar or navigation arrows
   - View AI-generated speaker notes
   - Listen to text-to-speech narration
   - Generate images for visual placeholders

3. **Save and Export**:
   - Sign in with Google to save presentations to the cloud
   - Export to PowerPoint (.pptx) format
   - Print as PDF

## Project Structure

```
vite-presentation-app/
├── public/
├── src/
│   ├── App.jsx          # Main application component
│   ├── main.jsx         # Application entry point
│   └── index.css        # Global styles and Tailwind imports
├── index.html           # HTML template
├── package.json         # Dependencies and scripts
├── vite.config.js       # Vite configuration
├── tailwind.config.js   # Tailwind CSS configuration
└── postcss.config.js    # PostCSS configuration
```

## Technologies Used

- **React 18**: UI framework
- **Vite**: Build tool and dev server
- **Tailwind CSS**: Utility-first CSS framework
- **Firebase**: Authentication and cloud storage (Firestore optimized for 1MB document limit)
- **Lucide React**: Icon library
- **Google Gemini API**: AI content generation and text-to-speech
- **Pixabay API**: High-quality stock images with CORS support
- **PptxGenJS**: PowerPoint export functionality with embedded images

## API Models & Services Used

- **Gemini 2.5 Flash Preview**: Content generation and structuring
- **Gemini 2.5 Flash TTS**: Text-to-speech narration for speaker coaching
- **Pixabay API**: Stock photo service (100 requests/minute free tier)

## Key Code Architecture

### Firebase Optimization for Document Size

The application implements an intelligent storage strategy to stay within Firestore's 1MB document size limit:

**Problem**: Base64-encoded images in presentations were causing documents to exceed the 1MB limit (reaching up to 1.6MB).

**Solution** (App.jsx:1263-1279):
```javascript
// Store only URLs in Firestore, not base64 data
const serializedPresentation = {
  meta: presentation.meta,
  slides: presentation.slides.map(slide => ({
    type: slide.type,
    title: slide.title,
    content: JSON.stringify(slide.content || []),
    imgData: slide.imgData || '' // URL only
    // imgBase64 is NOT stored - converted on-the-fly during export
  })),
  userId: user.uid,
  createdAt: serverTimestamp()
};
```

**Result**: Document size reduced from 1.6MB to ~150KB, eliminating Firestore save errors.

### On-the-Fly Image Conversion for PPTX Export

Images are converted to base64 only during PowerPoint export, not during storage:

**Helper Function** (App.jsx:335-350):
```javascript
const urlToBase64 = async (url) => {
  const response = await fetch(url);
  const blob = await response.blob();
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onloadend = () => resolve(reader.result);
    reader.onerror = reject;
    reader.readAsDataURL(blob);
  });
};
```

**PPTX Export with Async Image Processing** (App.jsx:1044-1096):
```javascript
// Changed from forEach to for loop to support async/await
for (let index = 0; index < presentation.slides.length; index++) {
  const slide = presentation.slides[index];

  if (slide.imgData) {
    // Convert URL to base64 on-the-fly
    const imgBase64 = await urlToBase64(slide.imgData);
    slideObj.addImage({
      data: imgBase64,
      x: 0, y: 0, w: '100%', h: '100%',
      sizing: { type: 'cover' }
    });
  }
}
```

### Pixabay Integration for Stock Images

The application uses Pixabay's CORS-friendly API to fetch high-quality stock images:

**Image Generation** (App.jsx:281-333):
```javascript
const generateSlideImage = async (prompt) => {
  const pixabayApiKey = import.meta.env.VITE_PIXABAY_API_KEY;

  // Extract keywords from AI-generated prompt
  const keywords = prompt
    .toLowerCase()
    .replace(/professional|presentation|slide|corporate/gi, '')
    .trim()
    .substring(0, 100);

  const searchUrl = `https://pixabay.com/api/?key=${pixabayApiKey}&q=${encodeURIComponent(keywords)}&image_type=photo&per_page=3&safesearch=true&orientation=horizontal&order=popular`;

  const response = await fetch(searchUrl);
  const data = await response.json();

  // Return URL only, not base64
  return {
    url: data.hits[0].largeImageURL,
    source: 'pixabay'
  };
};
```

### Title Slide Background Images

Title slides display background images with a semi-transparent blue overlay for visual consistency:

**Component Styling** (App.jsx:470-478):
```javascript
<div
  className="text-white flex-grow flex flex-col justify-center items-center text-center py-12"
  style={data.imgData ? {
    backgroundImage: `linear-gradient(rgba(30, 58, 138, 0.7), rgba(30, 58, 138, 0.7)), url(${data.imgData})`,
    backgroundSize: 'cover',
    backgroundPosition: 'center'
  } : { backgroundColor: '#1E3A8A' }}
>
```

This ensures consistent appearance between web display and PowerPoint export.

## Deployment to Netlify

This application is deployed on Netlify at [https://power-point-presentation.netlify.app/](https://power-point-presentation.netlify.app/)

### Deployment Steps:

1. **Build the production version**:
   ```bash
   npm run build
   ```

2. **Deploy to Netlify**:
   - Connect your GitHub repository to Netlify
   - Set build command: `npm run build`
   - Set publish directory: `dist`
   - Add environment variables in Netlify dashboard:
     - `VITE_GEMINI_API_KEY`
     - `VITE_GEMINI_MODEL`
     - `VITE_PIXABAY_API_KEY`

3. **Configure redirects** (optional):
   Create a `public/_redirects` file:
   ```
   /* /index.html 200
   ```

### Environment Variables on Netlify

⚠️ **Important**: Add all environment variables from your `.env` file to the Netlify dashboard under Site Settings → Environment Variables. Never commit the `.env` file to Git.

## Performance Optimizations

- **Lazy Loading**: Images loaded on-demand during generation
- **Client-Side Processing**: Runs fully client-side, no backend required
- **Optimized Storage**: URLs stored in Firestore, base64 conversion only during export
- **Code Splitting**: Vite automatically splits code for optimal loading
- **CDN Delivery**: PptxGenJS loaded from CDN only when needed for export

## Browser Compatibility

- Chrome/Edge (recommended)
- Firefox
- Safari
- Modern browsers with ES6+ support

## Troubleshooting

### Firebase Save Errors
If you encounter "document size exceeds limit" errors:
- The app now stores only URLs, not base64 data
- Images convert to base64 only during PPTX export
- Document size should be under 200KB

### CORS Issues with Images
- Pixabay API is CORS-friendly and should work without issues
- Images are fetched client-side and convert to base64 for PPTX embedding

### PowerPoint Export Issues
- Ensure images are loaded before exporting
- Check browser console for image conversion errors
- Large presentations may take 10-30 seconds to export

## License

This project is part of the Presentation-Generator repository.

## Support

For questions or feedback, contact:
- Twitter: @HnkHorizon
- YouTube: @HNK2005
- Instagram: hhnk.3693

## Source Code

GitHub: [https://github.com/HorizonHnk/Presentation-Generator.git](https://github.com/HorizonHnk/Presentation-Generator.git)

## Version

v3.0.0 - Cloud Sync Edition
