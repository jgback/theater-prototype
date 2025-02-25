import streamlit as st
import qrcode
from io import BytesIO
from PIL import Image

# Set page title
st.set_page_config(page_title="Theater Membership & Booking", page_icon="🎭")

# Ensure session state has a default step
if "step" not in st.session_state:
    st.session_state["step"] = "welcome"

# Debugging Output (TEMP: Remove later)
st.write(f"🔍 Debug: Current Step -> {st.session_state['step']}")

# ---- Welcome Screen ----
if st.session_state["step"] == "welcome":
    st.title("🎭 Welcome to the Theater Membership Prototype")
    st.write("Are you a new user or an existing member?")

    col1, col2 = st.columns(2)

    with col1:
        if st.button("🆕 New User"):
            st.session_state["step"] = "membership"
            st.rerun()

    with col2:
        if st.button("👤 Existing User"):
            st.session_state["step"] = "account_dashboard"
            st.rerun()

    st.stop()

# ---- Membership Signup ----
elif st.session_state["step"] == "membership":
    st.title("🎭 Theater Membership Signup")

    membership_options = {
        "Base Membership": {"price": "$99/year", "shows_allowed": 3, "perks": ["✔ Discounted extra tickets", "✔ Early booking access"]},
        "Premium Membership": {"price": "$199/year", "shows_allowed": 6, "perks": ["✔ Priority seating", "✔ Free concession per show", "✔ Bring a guest once per season"]}
    }

    col1, col2 = st.columns(2)
    with col1:
        st.write("### 🎟️ Base Membership")
        st.write(f"💰 **{membership_options['Base Membership']['price']}**")
        st.write(f"🎭 **Includes:** {membership_options['Base Membership']['shows_allowed']} shows")
        for perk in membership_options["Base Membership"]["perks"]:
            st.write(perk)
        base_selected = st.button("Choose Base Membership")

    with col2:
        st.write("### ⭐ Premium Membership")
        st.write(f"💰 **{membership_options['Premium Membership']['price']}**")
        st.write(f"🎭 **Includes:** {membership_options['Premium Membership']['shows_allowed']} shows")
        for perk in membership_options["Premium Membership"]["perks"]:
            st.write(perk)
        premium_selected = st.button("Choose Premium Membership")

    if base_selected or premium_selected:
        st.session_state["membership"] = "Base Membership" if base_selected else "Premium Membership"
        st.session_state["shows_remaining"] = membership_options[st.session_state["membership"]]["shows_allowed"]
        st.session_state["step"] = "theater_selection"
        st.rerun()

# ---- Home Theater Selection ----
elif st.session_state["step"] == "theater_selection":
    st.title("🏛 Select Your Home Theater")
    st.write("Choose one theater as your home base for membership perks.")

    theaters = {
        "Guthrie Theater": "📍 Minneapolis, MN\n🎭 Renowned regional theater with iconic productions.",
        "Steppenwolf Theatre": "📍 Chicago, IL\n🎭 Famous for cutting-edge contemporary plays.",
        "Berkeley Rep": "📍 Berkeley, CA\n🎭 Known for launching Broadway-bound shows."
    }

    selected_theater = st.radio("Select your home theater:", list(theaters.keys()))

    # Show theater details
    st.write(f"### {selected_theater}")
    st.write(theaters[selected_theater])

    if st.button("Confirm Home Theater"):
        st.session_state["home_theater"] = selected_theater
        st.session_state["step"] = "account_dashboard"
        st.rerun()

# ---- Account Dashboard ----
elif st.session_state["step"] == "account_dashboard":
    st.title("👤 My Account Dashboard")

    st.subheader("📅 Upcoming Events")
    if "booked_shows" not in st.session_state:
        st.session_state["booked_shows"] = []  # Initialize booked shows list

    if st.session_state["booked_shows"]:
        for show in st.session_state["booked_shows"]:
            st.write(f"🎭 **{show.get('title', '')}** - {show.get('date', '')} at {show.get('time', '')} @ {show.get('theater', '')}")
        if st.button("View Event Details"):
            st.session_state["step"] = "event_home"
            st.rerun()
    else:
        st.warning("❌ No events booked yet.")

    st.subheader("🎭 Browse & Book More Shows")
    if st.button("Find Shows at Home Theater"):
        st.session_state["step"] = "show_discovery"
        st.session_state["show_filter"] = "home"
        st.rerun()

    if st.button("Find Shows at Other Theaters"):
        st.session_state["step"] = "show_discovery"
        st.session_state["show_filter"] = "other"
        st.rerun()

    st.subheader("🎟️ Membership Status")
    st.write(f"🏛 **Home Theater:** {st.session_state.get('home_theater', 'Not Selected')}")
    st.write(f"🎭 **Shows Remaining:** {st.session_state.get('shows_remaining', 0)}")

    if st.button("Reset & Start Over"):
        st.session_state.clear()
        st.session_state["step"] = "welcome"
        st.rerun()

# ---- Show Discovery ----
elif st.session_state["step"] == "show_discovery":
    show_filter = st.session_state.get("show_filter", "home")

    if show_filter == "home":
        st.title(f"🎭 Shows at {st.session_state['home_theater']}")
    else:
        st.title("🎭 Shows at Other Theaters")

    # Sample shows for each theater
    sample_shows = {
        "Guthrie Theater": [{"title": "Hamlet", "date": "2025-03-10", "time": "7:00 PM"}],
        "Steppenwolf Theatre": [{"title": "True West", "date": "2025-03-12", "time": "7:30 PM"}],
        "Berkeley Rep": [{"title": "Hadestown", "date": "2025-03-14", "time": "7:00 PM"}]
    }

    if show_filter == "home":
        theater_shows = sample_shows.get(st.session_state["home_theater"], [])
    else:
        theater_shows = []
        for theater, shows in sample_shows.items():
            if theater != st.session_state["home_theater"]:  # Exclude home theater
                for show in shows:
                    show["theater"] = theater  # Add theater info to show
                    theater_shows.append(show)

    for show in theater_shows:
        st.write(f"🎭 **{show['title']}** - {show['date']} at {show['time']} @ {show.get('theater', st.session_state['home_theater'])}")
        if st.button(f"Book '{show['title']}'"):
            st.session_state["booked_shows"].append(show)
            st.session_state["step"] = "perk_selection"
            st.rerun()
    
    if st.button("Back to Dashboard"):
        st.session_state["step"] = "account_dashboard"
        st.rerun()

# ---- Perks Selection ----
elif st.session_state["step"] == "perk_selection":
    st.title("🍿 Add Perks")
    perks = {"snack": st.checkbox("🍿 Popcorn"), "drink": st.checkbox("🥤 Drink"), "parking": st.checkbox("🚗 Parking")}

    if st.button("Confirm Booking"):
        st.session_state["perks"] = perks
        st.session_state["step"] = "booking_confirmation"
        st.rerun()

# ---- Booking Confirmation ----
elif st.session_state["step"] == "booking_confirmation":
    st.title("🎟️ Booking Confirmation")
    st.write("✅ Your booking is confirmed!")
    if st.button("Go to Event Home"):
        st.session_state["step"] = "event_home"
        st.rerun()

# ---- Event Home Page (With Perks & QR Codes) ----
elif st.session_state["step"] == "event_home":
    st.title("🏛 Event Home Page")

    show = st.session_state.get("booked_show", {})
    perks = st.session_state.get("perks", {})

    st.write(f"### 🎭 {show.get('title', '')}")
    st.write(f"📅 **Date:** {show.get('date', '')} | ⏰ **Time:** {show.get('time', '')}")
    st.write(f"🏛 **Venue:** {st.session_state.get('home_theater', 'Unknown Theater')}")

    # Generate a QR code for the ticket
    ticket_data = f"Ticket for {show.get('title', '')} at {show.get('time', '')} on {show.get('date', '')}"
    qr = qrcode.make(ticket_data)
    qr_bytes = BytesIO()
    qr.save(qr_bytes, format="PNG")

    st.write("### 🎟️ Your Ticket QR Code:")
    st.image(qr_bytes.getvalue(), caption="Scan this for event entry", use_container_width=False)

    # Display selected perks
    st.write("### 🎟️ Your Perks:")
    perk_codes = []
    if perks.get("snack"):
        perk_codes.append("🍿 Popcorn ✅")
    if perks.get("drink"):
        perk_codes.append("🥤 Drink ✅")
    if perks.get("parking"):
        perk_codes.append("🚗 Parking ✅")

    if not perk_codes:
        st.write("❌ No perks selected.")
    else:
        for perk in perk_codes:
            st.write(perk)

    # Generate QR Codes for Perks
    st.write("### 📱 QR Codes for Perks:")
    for perk in perk_codes:
        perk_qr = qrcode.make(perk)
        perk_qr_bytes = BytesIO()
        perk_qr.save(perk_qr_bytes, format="PNG")
        st.image(perk_qr_bytes.getvalue(), caption=f"Scan for {perk}", use_container_width=False)

    # Modify Perks
    st.write("### Modify Your Perks:")
    snack = st.checkbox("🍿 Add/Remove Popcorn ($5)", value=perks.get("snack", False))
    drink = st.checkbox("🥤 Add/Remove Drink ($4)", value=perks.get("drink", False))
    parking = st.checkbox("🚗 Add/Remove Parking ($10)", value=perks.get("parking", False))

    if st.button("Update Perks"):
        st.session_state["perks"] = {"snack": snack, "drink": drink, "parking": parking}
        st.success("✅ Your perks have been updated!")
        st.rerun()

    # Return to Account Dashboard
    if st.button("Go to My Account"):
        st.session_state["step"] = "account_dashboard"
        st.rerun()