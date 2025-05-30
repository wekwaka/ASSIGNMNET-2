import math

def compute_join_from_yx():
    """
    Computes the horizontal distance and whole circle bearing between two points.
    Input coordinates are Y (Northing) and X (Easting).
    """
    print("\n--- Compute a Join (Y, X Coordinates) ---")

    try:
        # Input coordinates for Point 1
        y1 = float(input("Enter Y1 (Northing) of Point 1: "))
        x1 = float(input("Enter X1 (Easting) of Point 1: "))

        # Input coordinates for Point 2
        y2 = float(input("Enter Y2 (Northing) of Point 2: "))
        x2 = float(input("Enter X2 (Easting) of Point 2: "))

    except ValueError:
        print("Invalid input. Please enter numeric values for coordinates.")
        return

    # Calculate Delta Y (Northing) and Delta X (Easting)
    dY = y2 - y1
    dX = x2 - x1

    # Calculate Horizontal Distance (HD)
    hd = math.sqrt(dY**2 + dX**2)

    # Calculate Whole Circle Bearing (WCB)
    wcb_degrees = 0.0 # Initialize WCB_DEGREES

    if hd == 0:
        wcb_degrees = "Undefined (points are coincident)"
    else:
        # Handle cardinal directions first to avoid division by zero
        if dX == 0: # Line is due North or South
            if dY > 0:
                wcb_degrees = 0.0  # Due North
            elif dY < 0:
                wcb_degrees = 180.0 # Due South
        elif dY == 0: # Line is due East or West
            if dX > 0:
                wcb_degrees = 90.0  # Due East
            elif dX < 0:
                wcb_degrees = 270.0 # Due West
        else:
            # Calculate the reference angle (always positive, acute)
            # Uses atan(opposite/adjacent) -> atan(dX/dY) for angle from Y-axis (North)
            ref_angle_rad = abs(math.atan(dX / dY))
            ref_angle_deg = math.degrees(ref_angle_rad)

            # Determine WCB based on quadrant
            if dY > 0 and dX > 0: # Quadrant I (NE)
                wcb_degrees = ref_angle_deg
            elif dY < 0 and dX > 0: # Quadrant II (SE)
                wcb_degrees = 180.0 - ref_angle_deg
            elif dY < 0 and dX < 0: # Quadrant III (SW)
                wcb_degrees = 180.0 + ref_angle_deg
            elif dY > 0 and dX < 0: # Quadrant IV (NW)
                wcb_degrees = 360.0 - ref_angle_deg

    print(f"\nResults:")
    print(f"  Horizontal Distance (HD): {hd:.3f} meters")
    # Only format WCB if it's a number, otherwise print the string "Undefined"
    if isinstance(wcb_degrees, (int, float)):
        print(f"  Whole Circle Bearing (WCB): {wcb_degrees:.3f} degrees")
    else:
        print(f"  Whole Circle Bearing (WCB): {wcb_degrees}")
    print("-" * 30)


def compute_reduced_levels_hpc():
    """
    Computes reduced levels using the Height of Collimation (HPC) method.
    Handles Benchmark, Backsight, Intermediate Sights, Foresights, and Change Points.
    """
    print("\n--- Compute Reduced Levels (HPC Method) ---")

    try:
        rl_bm = float(input("Enter Reduced Level of Benchmark (RL_BM): "))
        bs_bm = float(input("Enter Backsight (BS) reading on Benchmark: "))
    except ValueError:
        print("Invalid input. Please enter numeric values for RL and BS.")
        return

    # Initialize lists to store all BS and FS readings for the arithmetic check
    all_bs_readings = [bs_bm]
    all_fs_readings = []
    first_rl = rl_bm # Store the very first RL for the final check
    last_rl = rl_bm # This will be updated with the RL of the last point observed

    # Calculate initial Height of Collimation
    hpc = rl_bm + bs_bm
    print(f"\nCalculated Height of Collimation (HPC): {hpc:.3f} meters")

    print("\n--- Enter Staff Readings (IS/FS) ---")
    print("  Type 'is' for Intermediate Sight.")
    print("  Type 'fs' for Foresight (at the end of current setup).")
    print("  Type 'done' to finish the entire leveling run.")


    while True:
        reading_type_input = input("Enter reading type (is/fs/done): ").strip().lower()

        if reading_type_input == 'done':
            # Ensure there's at least one FS to close the loop for arithmetic check
            if not all_fs_readings:
                print("Warning: No Foresight readings recorded. Arithmetic check may not be meaningful.")
            break # Exit the main loop for the entire leveling process

        try:
            reading_value = float(input(f"Enter {reading_type_input.upper()} reading: "))

            if reading_type_input == 'is':
                rl_point = hpc - reading_value
                print(f"  Reduced Level for this IS point: {rl_point:.3f} meters")
                last_rl = rl_point # Update last_rl with the most recent calculated RL

            elif reading_type_input == 'fs':
                all_fs_readings.append(reading_value)
                rl_point = hpc - reading_value
                print(f"  Reduced Level for this FS point (potential CP): {rl_point:.3f} meters")
                last_rl = rl_point # Update last_rl with the most recent calculated RL

                # Handle Change Point logic
                move_instrument = input("Move instrument to new setup? (yes/no): ").strip().lower()
                if move_instrument == 'yes':
                    new_bs = float(input(f"Enter New Backsight (BS) reading on current point (RL={rl_point:.3f}): "))
                    all_bs_readings.append(new_bs)
                    hpc = rl_point + new_bs # Calculate new HPC for the new setup
                    print(f"  New Height of Collimation (HPC): {hpc:.3f} meters")
                elif move_instrument == 'no':
                    print("Staying at current instrument setup.")
                else:
                    print("Invalid input for moving instrument. Assuming 'no'.")

            else:
                print("Invalid reading type. Please enter 'is', 'fs', or 'done'.")

        except ValueError:
            print("Invalid reading. Please enter a numeric value.")
        except Exception as e:
            print(f"An unexpected error occurred: {e}")


    print("\n--- Reduced Level Computations Complete ---")

    # Perform Arithmetic Check
    sum_bs = sum(all_bs_readings)
    sum_fs = sum(all_fs_readings)

    arithmetic_check_diff = sum_bs - sum_fs
    rl_diff = last_rl - first_rl

    print(f"\nArithmetic Check:")
    print(f"  Sum of Backsights (ΣBS): {sum_bs:.3f}")
    print(f"  Sum of Foresights (ΣFS): {sum_fs:.3f}")
    print(f"  ΣBS - ΣFS: {arithmetic_check_diff:.3f}")
    print(f"  Last RL ({last_rl:.3f}) - First RL ({first_rl:.3f}): {rl_diff:.3f}")

    # Allow for a small tolerance due to floating point arithmetic
    if abs(arithmetic_check_diff - rl_diff) < 0.001:
        print("Arithmetic check PASSED. Calculations are consistent.")
    else:
        print("Arithmetic check FAILED. There might be an error in calculations.")

    print("-" * 30)


def main_menu():
    """
    Provides a main menu for the survey operations application.
    """
    while True:
        print("\n=== Geomatics Survey Operations Application ===")
        print("1. Compute a Join")
        print("2. Compute Reduced Levels (HPC Method)")
        print("3. Exit")
        choice = input("Enter your choice (1-3): ").strip()

        if choice == '1':
            compute_join_from_yx()
        elif choice == '2':
            compute_reduced_levels_hpc()
        elif choice == '3':
            print("Exiting application. Goodbye!")
            break
        else:
            print("Invalid choice. Please enter 1, 2, or 3.")

# This ensures the main_menu() function runs when the script is executed
if __name__ == "__main__":
    main_menu()
