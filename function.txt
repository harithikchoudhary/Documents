def create_accuracy_assessment_prompt(source_code, business_requirements="", technical_requirements=""):
    """Create prompt for generating structured program documentation from source code"""
    return f"""
**PROGRAM DOCUMENTATION CREATION TASK**

You are an expert program analyst and technical writer.  
Your task is to analyze the given source code and generate a **clear, complete, and structured documentation**.

**SOURCE CODE:**
{source_code}

**BUSINESS REQUIREMENTS (if any):**
{business_requirements}

**TECHNICAL REQUIREMENTS (if any):**
{technical_requirements}

**DOCUMENTATION STRUCTURE (MANDATORY):**

1. **Introduction**
   - Purpose of the Document
   - Program Overview (business context, role in system)

2. **Scope and Assumptions**
   - Scope (what the program includes/excludes)
   - Assumptions (if any)

3. **System Architecture**
   - High-Level Design (components, modules, interactions – diagrams if possible)
   - Environment and Platform Details (languages, frameworks, databases, tools)
   - Dependencies (external systems, libraries, interfaces)
   - Data Sources and Storage (files, DBs, queues)

4. **Functional Description**
   - Business Requirements (mapped to program functionality, if available)
   - Main Processes (step-by-step breakdown of functionality)
   - Modules and Subroutines (list of modules/classes/functions with their purposes)
   - User Interfaces (screens, APIs, endpoints, if applicable)

5. **Program Logic and Flow**
   - Control Flow (pseudocode or execution path)
   - Rules, Logic, Validations (business rules, error handling, validations, audit logging)
   - Data Transformations (how input is processed into output)

6. **Input and Output Specifications**
   - Inputs (formats, sources, parameters, examples)
   - Outputs (generated files, responses, DB updates with layouts/examples)

7. **Summary**
   - High-level recap of purpose, scope, key functionality, and recommendations

**GUIDELINES:**
- Make the document self-contained and explanatory.
- Write for architects, developers, analysts, and new team members.
- Focus on clarity, completeness, and business understanding.
- Provide examples and pseudocode where possible.
"""

def save_accuracy_assessment(project_id, assessment_data):
    """Save program documentation to local directory"""
    try:
        output_dir_path = os.path.join("output", "converted", project_id)
        os.makedirs(output_dir_path, exist_ok=True)

        # Save JSON version
        json_path = os.path.join(output_dir_path, "program_documentation.json")
        with open(json_path, "w", encoding="utf-8") as f:
            json.dump(assessment_data, f, indent=2, ensure_ascii=False)

        # Save Markdown version
        md_path = os.path.join(output_dir_path, "Program_Documentation.md")
        md_content = format_assessment_as_markdown(assessment_data)
        with open(md_path, "w", encoding="utf-8") as f:
            f.write(md_content)

        logger.info(f"Program documentation saved to: {json_path} and {md_path}")
        return json_path, md_path
    except Exception as e:
        logger.error(f"Error saving documentation: {str(e)}")
        return None, None

def format_assessment_as_markdown(assessment_data):
    """Format the program documentation (handles nested JSON)"""

    intro = assessment_data.get("Introduction", {})
    scope = assessment_data.get("Scope and Assumptions", {})
    arch = assessment_data.get("System Architecture", {})
    func = assessment_data.get("Functional Description", {})
    logic = assessment_data.get("Program Logic and Flow", {})
    io_specs = assessment_data.get("Input and Output Specifications", {})
    summary = assessment_data.get("Summary", {})

    # Handle lists like "Main Processes"
    main_processes = func.get("Main Processes", [])
    if isinstance(main_processes, list):
        main_processes_md = "\n".join([f"- {p}" for p in main_processes])
    else:
        main_processes_md = main_processes or "Not provided"

    # Handle dict like "Modules and Subroutines"
    modules = func.get("Modules and Subroutines", {})
    if isinstance(modules, dict):
        modules_md = "\n".join([f"- **{k}**: {v}" for k, v in modules.items()])
    else:
        modules_md = modules or "Not provided"

    return f"""# Program Documentation

## 1. Introduction
### Purpose of the Document
{intro.get("Purpose of the Document", "Not provided")}

### Program Overview
{intro.get("Program Overview", "Not provided")}

---

## 2. Scope and Assumptions
### Scope
{scope.get("Scope", "Not provided")}

### Assumptions
{scope.get("Assumptions", "None")}

---

## 3. System Architecture
### High-Level Design
{arch.get("High-Level Design", "Not provided")}

### Environment and Platform Details
{arch.get("Environment and Platform Details", "Not provided")}

### Dependencies
{arch.get("Dependencies", "Not provided")}

### Data Sources and Storage
{arch.get("Data Sources and Storage", "Not provided")}

---

## 4. Functional Description
### Business Requirements
{func.get("Business Requirements", "Not provided")}

### Main Processes
{main_processes_md}

### Modules and Subroutines
{modules_md}

### User Interfaces
{func.get("User Interfaces", "Not applicable")}

---

## 5. Program Logic and Flow
### Control Flow
{logic.get("Control Flow", "Not provided")}

### Rules, Logic, Validations
{logic.get("Rules, Logic, Validations", "Not provided")}

### Data Transformations
{logic.get("Data Transformations", "Not provided")}

---

## 6. Input and Output Specifications
### Inputs
{io_specs.get("Inputs", "Not provided")}

### Outputs
{io_specs.get("Outputs", "Not provided")}

---

## 7. Summary
{summary.get("High-level Recap", "No summary available")}

---
*Generated on {time.strftime('%Y-%m-%d %H:%M:%S')}*
"""
