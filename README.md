// tailwind.config.js (custom theme setup)
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  css: {
    preprocessorOptions: {
      scss: {},
    },
  },
});

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

// src/pages/Home.jsx
import React, { useState } from 'react';
import { motion } from 'framer-motion';
import axios from 'axios';

export default function Home() {
  const [ingredients, setIngredients] = useState('');
  const [recipes, setRecipes] = useState([]);
  const [loading, setLoading] = useState(false);

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

  return (
    <div className="min-h-screen bg-luxury-dark text-white font-body p-6">
      <motion.div
        initial={{ opacity: 0, y: 50 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.6 }}
        className="max-w-xl mx-auto backdrop-blur-sm bg-white/10 rounded-2xl shadow-card p-6"
      >
        <h1 className="text-4xl font-display text-luxury-light mb-4">
          🍽️ Luxury Recipe Maker
        </h1>
        <p className="text-lg">
          Enter your ingredients and discover elegant, AI-curated recipes based on what's in your kitchen.
        </p>
        <input
          type="text"
          value={ingredients}
          onChange={(e) => setIngredients(e.target.value)}
          placeholder="e.g. spinach, cheese, garlic"
          className="mt-4 w-full px-4 py-2 rounded-lg bg-white/10 border border-white/20 focus:outline-none focus:ring-2 focus:ring-luxury-light"
        />
        <button
          onClick={fetchRecipes}
          className="mt-4 w-full bg-luxury-light text-white font-semibold py-2 px-4 rounded-lg hover:bg-luxury dark:shadow-lg"
        >
          {loading ? 'Loading...' : 'Generate Recipe'}
        </button>
      </motion.div>

      <div className="mt-8 space-y-6 max-w-3xl mx-auto">
        {recipes.map((recipe, index) => (
          <motion.div
            key={index}
            initial={{ opacity: 0, y: 30 }}
            animate={{ opacity: 1, y: 0 }}
            transition={{ delay: index * 0.1 }}
            className="bg-white/10 rounded-2xl shadow-card p-6 backdrop-blur-sm border border-white/10"
          >
            <h2 className="text-2xl font-display text-luxury-light mb-3">{recipe.title}</h2>
            <ul className="list-disc list-inside text-white/90 space-y-1">
              {recipe.ingredients.map((item, i) => (
                <li key={i}>{item}</li>
              ))}
            </ul>
          </motion.div>
        ))}
      </div>
    </div>
  );
}
