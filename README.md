🚀 How to Use Xonix Config Validator (XCV)Xonix Config Validator (XCV) is designed to be integrated into your development workflow with minimal overhead. It follows three mandatory steps: Install, Define Schema, and Validate.Step 1: Install the XCV CLI Tool (Installation)XCV is distributed as an npm package. It is recommended to install it globally for easy access across all your projects.Bash# Option 1: Install globally (recommended for command line usage)
npm install -g xonix-config-validator

# Option 2: Install as a development dependency
npm install --save-dev xonix-config-validator
Step 2: Define the Zod Schema (The Blueprint)The schema tells XCV what your configuration must look like. You define data types, constraints, and default values using Zod within a TypeScript file.File: config.schema.ts (Example Blueprint)TypeScript// config.schema.ts
import { z } from "zod";

// This schema defines the mandatory structure and types
export const AppConfigSchema = z.object({
  // 1. Required key, must be a string, with a default value
  PORT: z.string().default("3000"),
  
  // 2. Must be one of the specified values (Enum)
  NODE_ENV: z.enum(["development", "staging", "production"]),
  
  // 3. Required key, and must be a string in a valid URL format
  DATABASE_URL: z.string().url(),
  
  // 4. Must be a string of at least 32 characters
  JWT_SECRET: z.string().min(32, "Secret must be at least 32 characters long!"),
  
  // 5. Nested objects are supported for complex configurations
  FEATURE_FLAGS: z.object({
    newDashboard: z.boolean().default(false), // boolean type
    betaFeature: z.boolean(),
  }),
});

// Optional: Export the TypeScript type for use in your backend code
export type AppConfig = z.infer<typeof AppConfigSchema>;

Step 3: Run the Validation Command (Execution)Once the schema is defined, you use the xcv validate command, specifying the configuration file you want to check.3.1 Core Command: xcv validateXCV automatically attempts to locate your schema file (e.g., config.schema.ts) and applies it to the provided configuration file.File TypeCommandPurpose.envxcv validate --envParses and validates the default .env file (most common use case).JSONxcv validate config.jsonValidates a standard JSON configuration file.YAMLxcv validate config.yaml --yamlValidates a YAML configuration file. (The --yaml flag is mandatory)Custom Schemaxcv validate config.json --schema ./prod/schema.tsUse this if your schema definition file is not in the default location.3.2 Essential Command Flags (Cheat Sheet)FlagFunctionalityUse Case--envActivates environment file parsing.xcv validate .env.prod --env--strictThrows an error if the config file contains keys not defined in the schema. (Best for production integrity)xcv validate config.json --strict--watchEnables live file change monitoring. Validation runs immediately upon saving the config file. (Excellent for development)xcv watch config.dev.jsongenerate-envGenerates a sample .env file populated with default values from your schema.xcv generate-env > .env.example4. Understanding Validation Output (Result)The output is designed for immediate readability and CI/CD compatibility.A. Success Output (Validation Passed)A successful validation results in a GREEN message and an exit code 0. This is the signal for your CI/CD pipeline to proceed with deployment.BashValidation passed! Config is perfect.
B. Failure Output (Validation Failed)A validation failure results in a RED error report detailing every specific issue, and an exit code 1.BashValidation failed:
Path: DATABASE_URL    → Invalid url
Path: JWT_SECRET      → String must contain at least 32 character(s)
This non-zero exit code (1) is the critical mechanism that stops any automated deployment, preventing the configuration bug from ever reaching your live environment.
