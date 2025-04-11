// vite.config.js (with Tailwind + PostCSS)
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  css: {
    postcss: './postcss.config.cjs',
  },
});

// postcss.config.cjs
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};

// tailwind.config.cjs
module.exports = {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      fontFamily: {
        display: ['"Playfair Display"', 'serif'],
        body: ['Inter', 'sans-serif'],
      },
      colors: {
        luxury: {
          DEFAULT: '#14532d',
          light: '#22c55e',
          dark: '#064e3b',
        },
      },
      boxShadow: {
        card: '0 10px 30px rgba(0, 0, 0, 0.1)',
      },
      backdropBlur: {
        sm: '4px',
      },
    },
  },
  plugins: [],
};

// src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import Home from './pages/Home';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <Home />
  </React.StrictMode>
);

// src/index.css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html {
    transition: background-color 0.5s, color 0.5s;
  }
  body.dark {
    background-color: #0f172a;
    color: #f8fafc;
  }
}

// src/pages/Home.jsx
import React, { useState, useEffect } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import axios from 'axios';

export default function Home() {
  const [ingredients, setIngredients] = useState('');
  const [recipes, setRecipes] = useState([]);
  const [loading, setLoading] = useState(false);
  const [darkMode, setDarkMode] = useState(false);

  const fetchRecipes = async () => {
    setLoading(true);
    try {
      const response = await axios.post('https://recipe-maker-api.vercel.app/api/recipes', { ingredients });
      setRecipes(response.data.recipes);
    } catch (error) {
      console.error('Error fetching recipes:', error);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    document.body.classList.toggle('dark', darkMode);
  }, [darkMode]);

  return (
    <div className="min-h-screen bg-luxury-dark text-white font-body p-6 overflow-hidden transition-colors duration-500">
      <motion.div
        initial={{ opacity: 0, y: 50 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.6 }}
        className="max-w-xl mx-auto backdrop-blur-sm bg-white/10 rounded-2xl shadow-card p-6"
      >
        <div className="flex justify-between items-center mb-4">
          <motion.h1
            initial={{ scale: 0.9, opacity: 0 }}
            animate={{ scale: 1, opacity: 1 }}
            transition={{ duration: 0.5 }}
            className="text-4xl font-display text-luxury-light"
          >
            🍽️ Luxury Recipe Maker
          </motion.h1>
          <button
            onClick={() => setDarkMode(!darkMode)}
            className="text-sm bg-white/10 text-white px-3 py-1 rounded-lg border border-white/20 hover:bg-white/20"
          >
            {darkMode ? 'Light Mode' : 'Dark Mode'}
          </button>
        </div>
        <p className="text-lg">
          Enter your ingredients and discover elegant, AI-curated recipes based on what's in your kitchen.
        </p>
        <motion.input
          initial={{ opacity: 0, x: -20 }}
          animate={{ opacity: 1, x: 0 }}
          transition={{ delay: 0.3 }}
          type="text"
          value={ingredients}
          onChange={(e) => setIngredients(e.target.value)}
          placeholder="e.g. spinach, cheese, garlic"
          className="mt-4 w-full px-4 py-2 rounded-lg bg-white/10 border border-white/20 focus:outline-none focus:ring-2 focus:ring-luxury-light"
        />
        <motion.button
          whileHover={{ scale: 1.05 }}
          whileTap={{ scale: 0.95 }}
          onClick={fetchRecipes}
          className="mt-4 w-full bg-luxury-light text-white font-semibold py-2 px-4 rounded-lg hover:bg-luxury dark:shadow-lg"
        >
          {loading ? 'Loading...' : 'Generate Recipe'}
        </motion.button>
      </motion.div>

      <AnimatePresence>
        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          className="mt-8 space-y-6 max-w-3xl mx-auto"
        >
          {recipes.map((recipe, index) => (
            <motion.div
              key={index}
              initial={{ opacity: 0, y: 30 }}
              animate={{ opacity: 1, y: 0 }}
              transition={{ delay: index * 0.1 }}
              className="bg-white/10 rounded-2xl shadow-card p-6 backdrop-blur-sm border border-white/10"
            >
              <motion.h2
                initial={{ x: -10, opacity: 0 }}
                animate={{ x: 0, opacity: 1 }}
                transition={{ delay: index * 0.1 + 0.2 }}
                className="text-2xl font-display text-luxury-light mb-3"
              >
                {recipe.title}
              </motion.h2>
              <ul className="list-disc list-inside text-white/90 space-y-1">
                {recipe.ingredients.map((item, i) => (
                  <motion.li
                    key={i}
                    initial={{ x: -10, opacity: 0 }}
                    animate={{ x: 0, opacity: 1 }}
                    transition={{ delay: i * 0.05 }}
                  >
                    {item}
                  </motion.li>
                ))}
              </ul>
            </motion.div>
          ))}
        </motion.div>
      </AnimatePresence>
    </div>
  );
}
