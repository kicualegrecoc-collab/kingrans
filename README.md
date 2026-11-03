# Students Consultation Scheduler System
# PHINMA Cagayan de Oro College

import os
from datetime import datetime

USERS = "users.txt"
SCHEDULES = "schedules.txt"
APPOINTMENTS = "appointments.txt"


def read_file(filename):
    if not os.path.exists(filename):
        open(filename, "w").close()

    with open(filename, "r") as file:
        return [line.strip() for line in file if line.strip()]


def register():
    name = input("Full Name: ")
    username = input("Username: ")
    password = input("Password: ")
    role = input("Role (student/teacher): ").lower()

    if role not in ["student", "teacher"]:
        print("Invalid role!")
        return

    users = read_file(USERS)

    if any(u.split("|")[1] == username for u in users):
        print("Username already exists!")
        return

    with open(USERS, "a") as file:
        file.write(f"{name}|{username}|{password}|{role}\n")

    print("Registration successful!")


def login():
    username = input("Username: ")
    password = input("Password: ")

    for user in read_file(USERS):
        name, user_name, user_pass, role = user.split("|")

        if username == user_name and password == user_pass:
            print(f"\nWelcome, {name}!")
            if role == "teacher":
                teacher_menu(username)
            else:
                student_menu(username)
            return

    print("Invalid username or password!")


def teacher_menu(username):
    while True:
        print("\n=== TEACHER MENU ===")
        print("1. Set Schedule")
        print("2. View My Schedules")
        print("3. View Appointments")
        print("4. Logout")

        choice = input("Choose: ")

        if choice == "1":
            day = input("Day (Monday-Sunday): ").capitalize()

            if day not in [
                "Monday", "Tuesday", "Wednesday", "Thursday",
                "Friday", "Saturday", "Sunday"
            ]:
                print("Invalid day!")
                continue

            start = input("Start Time (e.g. 8:00 AM): ")
            end = input("End Time (e.g. 11:30 AM): ")

            try:
                start_time = datetime.strptime(start, "%I:%M %p")
                end_time = datetime.strptime(end, "%I:%M %p")

                if start_time >= end_time:
                    print("End time must be later than start time!")
                    continue
            except ValueError:
                print("Invalid time format!")
                continue

            with open(SCHEDULES, "a") as file:
                file.write(f"{username}|{day}|{start}|{end}\n")

            print("Schedule added successfully!")

        elif choice == "2":
            for s in read_file(SCHEDULES):
                teacher, day, start, end = s.split("|")
                if teacher == username:
                    print(f"{day}: {start} - {end}")

        elif choice == "3":
            for a in read_file(APPOINTMENTS):
                student, teacher, day, time, reason = a.split("|")
                if teacher == username:
                    print(
                        f"Student: {student} | {day} at {time}"
                        f" | Reason: {reason}"
                    )

        elif choice == "4":
            break

        else:
            print("Invalid choice!")


def student_menu(username):
    while True:
        print("\n=== STUDENT MENU ===")
        print("1. View Available Schedules")
        print("2. Book Consultation")
        print("3. View My Appointments")
        print("4. Logout")

        choice = input("Choose: ")

        if choice == "1":
            for s in read_file(SCHEDULES):
                teacher, day, start, end = s.split("|")
                print(f"Teacher: {teacher} | {day}: {start} - {end}")

        elif choice == "2":
            schedules = read_file(SCHEDULES)

            if not schedules:
                print("No schedules available.")
                continue

            teacher = input("Teacher Username: ")
            day = input("Day: ").capitalize()
            time = input("Consultation Time (e.g. 9:00 AM): ")
            reason = input("Reason for consultation: ")

            available = False

            for s in schedules:
                t, d, start, end = s.split("|")

                if t == teacher and d == day:
                    try:
                        chosen = datetime.strptime(time, "%I:%M %p")
                        start_time = datetime.strptime(start, "%I:%M %p")
                        end_time = datetime.strptime(end, "%I:%M %p")

                        if start_time <= chosen < end_time:
                            available = True
                    except ValueError:
                        pass

            if available:
                with open(APPOINTMENTS, "a") as file:
                    file.write(
                        f"{username}|{teacher}|{day}|{time}|{reason}\n"
                    )
                print("Consultation booked successfully!")
            else:
                print("Invalid teacher, day, or time.")

        elif choice == "3":
            for a in read_file(APPOINTMENTS):
                student, teacher, day, time, reason = a.split("|")
                if student == username:
                    print(
                        f"Teacher: {teacher} | {day} at {time}"
                        f" | Reason: {reason}"
                    )

        elif choice == "4":
            break

        else:
            print("Invalid choice!")


# MAIN PROGRAM
while True:
    print("\n=== CONSULTATION SCHEDULER ===")
    print("1. Register")
    print("2. Login")
    print("3. Exit")

    choice = input("Choose: ")

    if choice == "1":
        register()
    elif choice == "2":
        login()
    elif choice == "3":
        print("Thank you for using the system!")
        break
    else:
        print("Invalid choice!")
