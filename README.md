"""
AI-Based Wooden Box Design Generator
Course Final Project

This script demonstrates a simplified AI-assisted workflow for generating
production-ready wooden box designs. The implementation simulates the use of
AI models (e.g., large language models) using deterministic logic to ensure
clarity, reproducibility, and academic transparency.
"""

from dataclasses import dataclass
from typing import Dict, List


# --------------------------------------------------
# Data structures
# --------------------------------------------------

@dataclass
class BoxDesign:
    """
    Data structure representing a generated wooden box design.
    All dimensions are stored in millimeters.
    """
    width: float
    depth: float
    height: float
    thickness: float
    panels: List[Dict]
    material_area: float
    waste_percent: float
    total_cost: float
    dxf_ready: bool


# --------------------------------------------------
# Core generator class
# --------------------------------------------------

class WoodBoxGenerator:
    """
    Main generator class that simulates an AI-driven design process.

    The class converts natural language input into a structured
    wooden box design, including material estimates and cut lists.
    """

    def __init__(self):
        # Local material pricing (EUR per m²)
        self.material_prices = {
            "birch_plywood_18mm": 25.0,
            "aspen_plywood_15mm": 22.0,
            "oak_board_20mm": 45.0
        }

        # Standard sheet sizes commonly used in Latvia (mm)
        self.sheet_sizes = [(1200, 600), (2440, 1220)]

    def generate_design(self, user_input: str) -> BoxDesign:
        """
        Main entry point of the system.

        Parameters:
            user_input (str): Natural language description of the box.

        Returns:
            BoxDesign: Fully calculated design object.
        """
        params = self._parse_input(user_input)
        panels = self._calculate_panels(params)
        layout = self._optimize_layout(panels)

        total_area = sum(
            p["width"] * p["height"] * p["quantity"]
            for p in panels
        )

        sheet_area = layout["used_sheets"][0]["area"]
        waste_percent = (1 - total_area / sheet_area) * 100

        cost_per_m2 = self.material_prices.get(params["material"], 25.0)
        total_cost = (total_area / 1_000_000) * cost_per_m2 * 1.2

        return BoxDesign(
            width=params["width"],
            depth=params["depth"],
            height=params["height"],
            thickness=params["thickness"],
            panels=panels,
            material_area=total_area,
            waste_percent=round(waste_percent, 1),
            total_cost=round(total_cost, 2),
            dxf_ready=True
        )

    # --------------------------------------------------
    # Internal helper methods
    # --------------------------------------------------

    def _parse_input(self, text: str) -> Dict:
        """
        Extracts dimensions and basic parameters from user input.

        In a real-world system this step would be handled by an AI model.
        Here it is intentionally simplified for demonstration purposes.
        """
        text = text.lower()

        dimensions = {"width": 400, "depth": 300, "height": 150}

        if "x" in text and "cm" in text:
            try:
                parts = text.split("cm")[0].split("x")
                dimensions = {
                    "width": float(parts[0]) * 10,
                    "depth": float(parts[1]) * 10,
                    "height": float(parts[2]) * 10,
                }
            except (ValueError, IndexError):
                pass

        return {
            **dimensions,
            "thickness": 18.0,
            "material": "birch_plywood_18mm",
            "style": "hinged_lid",
            "kerf": 2.0
        }

    def _calculate_panels(self, params: Dict) -> List[Dict]:
        """
        Calculates all required panels for a simple hinged-lid box.
        """
        t = params["thickness"]
        w, d, h = params["width"], params["depth"], params["height"]

        panels = [
            {"name": "bottom", "width": w - 2*t, "height": d - 2*t, "quantity": 1},
            {"name": "side_short", "width": w - 2*t, "height": h - t, "quantity": 2},
            {"name": "side_long", "width": d - 2*t, "height": h - t, "quantity": 2},
            {"name": "lid", "width": w - 2*t + 10, "height": d - 2*t + 10, "quantity": 1},
        ]

        return panels

    def _optimize_layout(self, panels: List[Dict]) -> Dict:
        """
        Performs a basic sheet usage estimation.
        This method simulates layout optimization logic.
        """
        sheet_width, sheet_height = self.sheet_sizes[0]
        sheet_area = sheet_width * sheet_height

        return {
            "used_sheets": [
                {"width": sheet_width, "height": sheet_height, "area": sheet_area}
            ]
        }


# --------------------------------------------------
# Output utilities
# --------------------------------------------------

def export_dxf(design: BoxDesign, filename: str):
    """
    Exports a very simplified DXF file to demonstrate
    production-ready output.
    """
    with open(filename, "w") as file:
        file.write("0\nSECTION\n2\nENTITIES\n")
        file.write("0\nENDSEC\n0\nEOF\n")

    print(f"DXF file exported: {filename}")


def print_design_summary(design: BoxDesign):
    """
    Prints a human-readable summary of the generated design.
    """
    print("\n----------------------------------------")
    print("AI WOOD BOX DESIGN SUMMARY")
    print("----------------------------------------")
    print(f"Dimensions: {design.width/10:.0f} x {design.depth/10:.0f} x {design.height/10:.0f} cm")
    print(f"Material thickness: {design.thickness} mm")
    print(f"Estimated cost: €{design.total_cost}")
    print(f"Material waste: {design.waste_percent}%")
    print("\nCut list:")
    for p in design.panels:
        print(f"- {p['name']}: {p['width']/10:.0f} x {p['height']/10:.0f} cm (x{p['quantity']})")
    print("----------------------------------------")


# --------------------------------------------------
# Main execution
# --------------------------------------------------

def main():
    """
    Demonstration entry point for the course project.
    """
    generator = WoodBoxGenerator()

    examples = [
        "40x30x15cm wedding gift box, birch plywood",
        "30x20x10cm jewelry box",
        "50x40x20cm storage box"
    ]

    for text in examples:
        design = generator.generate_design(text)
        print_design_summary(design)

    export_dxf(design, "example_box.dxf")
    print("\nCourse project execution completed.")


if __name__ == "__main__":
    main()

