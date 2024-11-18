#Initialize Project

npx create-react-app dynamic-form-generator --template typescript
cd dynamic-form-generator
npm install tailwindcss react-hook-form zod @hookform/resolvers react-split
npm install -D jest @testing-library/react @testing-library/jest-dom playwright
npx tailwindcss init

#tailwind.config.js

module.exports = {
  content: ["./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
index.css

css
Copy code
@tailwind base;
@tailwind components;
@tailwind utilities;

#JSONEditor.tsx
#Handles the JSON schema input with syntax validation.
import React, { useState } from "react";
import { z } from "zod";

const schemaValidator = z.object({
  formTitle: z.string(),
  formDescription: z.string(),
  fields: z.array(
    z.object({
      id: z.string(),
      type: z.string(),
      label: z.string(),
      required: z.boolean(),
      placeholder: z.string().optional(),
      options: z.array(z.object({ value: z.string(), label: z.string() })).optional(),
      validation: z.object({
        pattern: z.string(),
        message: z.string(),
      }).optional(),
    })
  ),
});

interface Props {
  onUpdateSchema: (schema: any) => void;
}

const JSONEditor: React.FC<Props> = ({ onUpdateSchema }) => {
  const [json, setJson] = useState<string>("");

  const handleInputChange = (e: React.ChangeEvent<HTMLTextAreaElement>) => {
    setJson(e.target.value);

    try {
      const parsed = JSON.parse(e.target.value);
      schemaValidator.parse(parsed);
      onUpdateSchema(parsed);
    } catch {
      // Handle errors in validation (optional)
    }
  };

  return (
    <textarea
      className="w-full h-full border p-2 rounded"
      value={json}
      onChange={handleInputChange}
      placeholder="Paste your JSON schema here..."
    />
  );
};

#export default JSONEditor;
#DynamicForm.tsx
#Renders the form based on the provided JSON schema.

import React from "react";
import { useForm } from "react-hook-form";

type Field = {
  id: string;
  type: string;
  label: string;
  required: boolean;
  placeholder?: string;
  options?: { value: string; label: string }[];
};

type FormSchema = {
  formTitle: string;
  formDescription: string;
  fields: Field[];
};

interface Props {
  schema: FormSchema;
}

const DynamicForm: React.FC<Props> = ({ schema }) => {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const onSubmit = (data: any) => {
    console.log("Form submitted:", data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <h2 className="text-xl font-bold">{schema.formTitle}</h2>
      <p className="text-gray-600">{schema.formDescription}</p>
      {schema.fields.map((field) => (
        <div key={field.id}>
          <label className="block font-medium">{field.label}</label>
          {field.type === "text" || field.type === "email" ? (
            <input
              {...register(field.id, { required: field.required })}
              placeholder={field.placeholder}
              className="w-full border p-2 rounded"
            />
          ) : null}
          {errors[field.id] && <p className="text-red-500">This field is required</p>}
        </div>
      ))}
      <button type="submit" className="bg-blue-500 text-white p-2 rounded">
        Submit
      </button>
    </form>
  );
};

#export default DynamicForm;
#FormPreview.tsx
#Displays the generated form in real-time.

import React from "react";
import DynamicForm from "./DynamicForm";

interface Props {
  schema: any;
}

const FormPreview: React.FC<Props> = ({ schema }) => (
  <div className="p-4">
    {schema ? <DynamicForm schema={schema} /> : <p>Enter a valid JSON schema to see the preview.</p>}
  </div>
);

#export default FormPreview;
#App.tsx
#Combines JSONEditor and FormPreview.

import React, { useState } from "react";
import JSONEditor from "./components/JSONEditor";
import FormPreview from "./components/FormPreview";

const App: React.FC = () => {
  const [schema, setSchema] = useState<any>(null);

  return (
    <div className="flex h-screen">
      <div className="w-1/2 p-4 border-r">
        <h1 className="text-xl font-bold mb-4">JSON Editor</h1>
        <JSONEditor onUpdateSchema={setSchema} />
      </div>
      <div className="w-1/2 p-4">
        <h1 className="text-xl font-bold mb-4">Form Preview</h1>
        <FormPreview schema={schema} />
      </div>
    </div>
  );
};

export default App;

