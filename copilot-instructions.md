 .github/copilot-instructions.md

# Bengali Land Deed Generator - Development Guidelines

## Project Context

**Full-Stack Bengali Land Deed Generator** (দলিল) is a mobile app(ios + android) for West Bengal land deed document generation with AI-powered PDF extraction, real-time preview, and multi-font customization.

## Tech Stack

### Frontend
- Next.js 14 with TypeScript
- React 18
- Tailwind CSS
- React Icons
- DocX (Word generation)
- html2canvas + jsPDF (Print)
- PWA with Service Worker

### BackendInstall: npm install tesseract.js
Then use it to extract Bengali text from PNG filesInstall: npm install tesseract.js
- Node.js + Express
- Anthropic Claude Sonnet 4 API
- Multer (File uploads)
- ES6 Modules

### Key Features
- 50+ automatic field extraction
- Bengali text conversion
- Real-time WYSIWYG preview
- Export to Word/PDF
- PWA for Android + ios installation
- Offline capability 

### PDF Extraction Capabilities

#### Multi-Format Support
- **PDF format**: E-Assessment slip (English document)
- **PNG/JPEG formats**: Scanned images with Bengali text (primary source for Bengali data)
- Processes multiple documents in a single upload
- Handles mixed format batches automatically

#### Extraction & Verification Logic

**When PNG document is uploaded:**
1. Extract all Bengali details from PNG document
2. Extract English details from PDF (E-Assessment slip)
3. Cross-verify PNG Bengali data with PDF English data
4. Apply validation rules below

**Priority for Bengali Text:**
- **PRIMARY**: Bengali text extracted from PNG document
- **SECONDARY**: English text from PDF converted to Bengali
- If PNG Bengali and converted Bengali don't match → Use PNG Bengali as final source
- Do NOT use converted/extracted Bengali text if it conflicts  with PNG Bengali version

**Critical Fields - Extract from PNG Bengali as Primary:**
- **Donor & Donee** details (দাতা ও গ্রহণকারী)
- **Seller & Buyer** details (বিক্রেতা ও ক্রেতা)
- **Property description** (সম্পত্তি বর্ণনা)
- **Transaction details** (লেনদেনের বিবরণ)

**Verification Rules:**
- Extract all mentioned details from both PNG and PDF
- Some details may exist in PNG but not in PDF (accept from PNG)
- Some details may exist in PDF but not in PNG (use from PDF)
- For overlapping details: PNG Bengali version takes precedence
- If data mismatch: Flag as conflict and use PNG Bengali value
- Ensure consistency between both sources before finalizing

## 📄 Updated Deed Generation Logic (Mobile-First)

### 1. Document Input & Priority Hierarchy

The system must handle two distinct input types with a strict priority sequence:

* **Primary Source (PNG/Image):** Bengali scanned images or photos. These contain the "Source of Truth" for names and handwritten details.
* **Secondary Source (PDF):** The E-Assessment slip (English). Used for structured data but must be cross-verified against the Primary Source.

### 2. The Three-Part Generation Pipeline

| Section | Focus Area | Extraction Logic |
| --- | --- | --- |
| **Part 1: Brief Info** | Land & Assessment | **Direct Mapping:** Extract plot numbers, khatian, and value directly from the E-Assessment slip PDF. No creative changes allowed. |
| **Part 2: Entity Details** | Buyer/Seller & Donor/Donee | **OCR Priority:** Extract names and addresses. If the PNG scan (Bengali) conflicts with the PDF (English), the **Bengali Unicode text from the scan MUST be used.** |
| **Part 3: Legal Body (Boyal)** | "Khash Jomi" to "Till Date" | **Dynamic Synthesis:** Replace the entire body section. Adjust legal phrasing based on the number of parties, relationship details, and dates. Phrasing must learn from the 450 legacy examples. |

### 3. High-Precision Property Schedule (70% Focus)

This is the most critical part of the document.

* **Tafsil (তফসিল):** Convert PDF table data into formal Bengali legal schedule format.
* **Chauhaddi (চৌহদ্দি):** The system must prompt the user for the four boundaries (North, South, East, West) and integrate them precisely into the text.
* **Verification:** AI must cross-verify that the total area and plot numbers in the generated Bengali text match the E-Assessment slip exactly.

### 4. Machine Learning & Training

* **Example-Based Learning:** The system uses 450 historical deeds as a "Knowledge Base" (via RAG) to learn regional Malda-specific legal phrasing.
* **Continuous Improvement:** The system must allow for manual corrections during the testing phase. These corrections should be stored to "train" the AI's future responses.

### 5. Technical Requirements (Cross-Platform)

* **Frontend:** React Native (Expo) for iOS/Android compatibility.
* **OCR Engine:** Tesseract.js for Bengali script recognition.
* **LLM:** Gemini 1.5 Pro or Claude Sonnet 4 via API for reasoning and synthesis.

## Development Workflow

### Key Principles
1. **Type Safety**: Use TypeScript throughout
2. **Component Reusability**: Keep components small and focused
3. **Error Handling**: Graceful fallbacks for all operations
4. **Performance**: Lazy load components, optimize images
5. **Accessibility**: ARIA labels, semantic HTML

### Folder Structure
```
frontend/
├── app/                    # Next.js app directory
├── components/             # React components
├── styles/                 # Global styles
├── public/                 # Static assets, PWA files
└── utils/                  # Helper functions

backend/
├── src/
│   ├── routes/            # API endpoints
│   ├── services/          # Business logic
│   ├── middleware/        # Express middleware
│   └── server.js          # Entry point
└── package.json
```

### Component Standards

1. **Use TypeScript interfaces**
```typescript
interface ComponentProps {
  deedData: DeedData;
  onUpdate: (field: string, value: string) => void;
}
```

2. **Separate concerns**
```
- UI rendering
- Logic/state management
- Data fetching
```

3. **Document complex logic**
```javascript
// Extract deed fields from PDF
// Priority: seller info > property > transaction
function extractFields(data) {
  // Detailed comment explaining extraction logic
}
```

## File Naming Conventions

- Components: PascalCase (`DeedPreview.tsx`)
- Utilities: camelCase (`extractData.ts`)
- Styles: kebab-case (`global-theme.css`)
- Constants: UPPER_SNAKE_CASE

## Common Tasks

### Adding a New Deed Field

1. Add to `DEED_FIELDS` in `backend/src/services/claudeService.js`
2. Update Claude prompt
3. Add input in `frontend/components/Sidebar.tsx` (Data tab)
4. Update template in `backend/src/services/deedTemplateService.js`
5. Test extraction and rendering

### Customizing the Deed Template

Edit `generateDeedHTML()` in `backend/src/services/deedTemplateService.js`
- Modify HTML structure
- Update CSS for layout
- Test print output
- Verify mobile rendering

### Adding Export Format

1. Create export service in `backend/src/services/`
2. Add API endpoint in `backend/src/routes/deed.js`
3. Add button in `frontend/app/page.tsx`
4. Connect to handler in frontend

### Testing New Feature

```bash
# Backend
npm run dev  # Runs on :5001

# Frontend (separate terminal)
npm run dev  # Runs on :3000

# Test in browser at http://localhost:3000
```

## Performance Guidelines

- Keep components under 300 lines
- Lazy load preview iframe
- Debounce style updates
- Cache API responses
- Optimize images

## Security Checklist

- [ ] API key in .env (never committed)
- [ ] Input validation on all fields
- [ ] File size limits enforced
- [ ] CORS properly configured
- [ ] No sensitive data in console
- [ ] HTTPS in production

## Code Quality

- Use ESLint for linting
- Format with Prettier
- Write meaningful commits
- Add JSDoc for public functions
- Test before committing

## Debugging Tips

1. **Network Issues**: Check DevTools Network tab
2. **State Problems**: Use React DevTools
3. **API Errors**: Log at each step
4. **Build Errors**: Check Console for full stack trace

## Useful Commands

```bash
# Backend
npm run dev          # Development with nodemon
npm run start        # Production

# Frontend
npm run dev          # Development
npm run build        # Production build
npm run lint         # ESLint check

# General
npm install          # Install dependencies
npm audit            # Check vulnerabilities
```

## Documentation

- **API**: Document in code comments
- **Components**: JSDoc with params
- **Setup**: See docs/SETUP.md
- **Architecture**: See docs/ARCHITECTURE.md

## When to Ask for Help

- Architectural decisions
- Security concerns
- Performance optimizations
- Complex debugging

---

**Version**: 1.0  
**Last Updated**: April 2026  
**Status**: Active Development
