import random

def get_match_details():
    """
    Collects match details from the user.
    """
    print("⚽ Welcome to the Advanced Match Prediction System!")
    
    # Collect team names
    home_team = input("🏟️ Enter the home team name: ").strip()
    away_team = input("🚩 Enter the away team name: ").strip()
    
    # Collect team strengths
    home_strength = float(input(f"💪 Enter {home_team}'s strength (1-10): ").strip())
    away_strength = float(input(f"💪 Enter {away_team}'s strength (1-10): ").strip())
    
    # Collect team ranks
    home_rank = int(input(f"📊 Enter {home_team}'s rank in the league: ").strip())
    away_rank = int(input(f"📊 Enter {away_team}'s rank in the league: ").strip())
    
    # Collect last 5 match results
    home_results = validate_results_input(f"📈 Enter the results for the last 5 matches of {home_team} (W=Win, D=Draw, L=Loss): ")
    away_results = validate_results_input(f"📈 Enter the results for the last 5 matches of {away_team} (W=Win, D=Draw, L=Loss): ")
    
    # Collect betting odds
    home_odds = float(input(f"💵 Enter the betting odds for {home_team} to win: ").strip())
    away_odds = float(input(f"💵 Enter the betting odds for {away_team} to win: ").strip())
    
    # Collect key player performance
    home_key_player = float(input(f"⭐ Performance of {home_team}'s key player (1-10): ").strip())
    away_key_player = float(input(f"⭐ Performance of {away_team}'s key player (1-10): ").strip())
    
    # Collect coach influence
    home_coach = float(input(f"🧠 Rate {home_team}'s coach influence (1-10): ").strip())
    away_coach = float(input(f"🧠 Rate {away_team}'s coach influence (1-10): ").strip())
    
    # Collect match importance
    importance = float(input("🔍 How important is this match? (1-10): ").strip())
    
    # New inputs
    pitch_quality = float(input("🌱 Rate the pitch quality (1-10): ").strip())
    home_readiness = float(input(f"📋 Rate {home_team}'s player readiness (1-10): ").strip())
    away_readiness = float(input(f"📋 Rate {away_team}'s player readiness (1-10): ").strip())
    home_absences = int(input(f"🩹 Number of key absences for {home_team}: ").strip())
    away_absences = int(input(f"🩹 Number of key absences for {away_team}: ").strip())
    
    return {
        "home_team": home_team,
        "away_team": away_team,
        "home_strength": home_strength,
        "away_strength": away_strength,
        "home_rank": home_rank,
        "away_rank": away_rank,
        "home_results": home_results,
        "away_results": away_results,
        "home_odds": home_odds,
        "away_odds": away_odds,
        "home_key_player": home_key_player,
        "away_key_player": away_key_player,
        "home_coach": home_coach,
        "away_coach": away_coach,
        "importance": importance,
        "pitch_quality": pitch_quality,
        "home_readiness": home_readiness,
        "away_readiness": away_readiness,
        "home_absences": home_absences,
        "away_absences": away_absences
    }

def validate_results_input(prompt):
    """
    Validates the format of the last 5 matches' results.
    """
    while True:
        results = input(prompt).strip().upper()
        if len(results) == 5 and all(r in {"W", "D", "L"} for r in results):
            return results
        else:
            print("❌ Invalid format! Please enter exactly 5 results using only W, D, or L (e.g., WWDDL).")

def calculate_form_factor(results):
    """
    Calculates a form factor based on the last 5 matches.
    W = 3 points, D = 1 point, L = 0 points.
    """
    score = 0
    for result in results:
        if result == "W":
            score += 3
        elif result == "D":
            score += 1
    return score / 15  # Normalize to a value between 0 and 1

def adjust_by_odds(score, odds):
    """
    Adjusts the score based on betting odds.
    Lower odds suggest a higher likelihood of winning.
    """
    return score * (1 / odds)

def predict_result(match_details):
    """
    Predicts the match result based on input details.
    """
    # Calculate form factors
    home_form = calculate_form_factor(match_details["home_results"])
    away_form = calculate_form_factor(match_details["away_results"])
    
    # Calculate rank influence (lower rank is better)
    home_rank_factor = max(0.1, 1 - (match_details["home_rank"] / 20))
    away_rank_factor = max(0.1, 1 - (match_details["away_rank"] / 20))
    
    # Calculate base scores
    home_score = (
        match_details["home_strength"] * 0.25 +
        home_form * 0.15 +
        home_rank_factor * 0.15 +
        match_details["home_key_player"] * 0.1 +
        match_details["home_coach"] * 0.1 +
        match_details["home_readiness"] * 0.1 -
        match_details["home_absences"] * 0.05 +
        match_details["pitch_quality"] * 0.1 +
        random.uniform(-0.5, 0.5)
    )
    away_score = (
        match_details["away_strength"] * 0.25 +
        away_form * 0.15 +
        away_rank_factor * 0.15 +
        match_details["away_key_player"] * 0.1 +
        match_details["away_coach"] * 0.1 +
        match_details["away_readiness"] * 0.1 -
        match_details["away_absences"] * 0.05 +
        match_details["pitch_quality"] * 0.1 +
        random.uniform(-0.5, 0.5)
    )
    
    # Adjust scores by betting odds
    home_score = adjust_by_odds(home_score, match_details["home_odds"])
    away_score = adjust_by_odds(away_score, match_details["away_odds"])
    
    # Adjust scores based on importance
    multiplier = match_details["importance"] / 10
    home_score *= multiplier
    away_score *= multiplier
    
    # Round scores to nearest integer
    home_score = max(0, round(home_score))
    away_score = max(0, round(away_score))
    
    # Calculate total goals and Odd/Even
    total_goals = home_score + away_score
    is_odd = total_goals % 2 != 0
    
    # Determine outcomes
    both_teams_to_score = home_score > 0 and away_score > 0
    
    result = {
        "home_team": match_details["home_team"],
        "away_team": match_details["away_team"],
        "home_score": home_score,
        "away_score": away_score,
        "total_goals": total_goals,
        "both_teams_to_score": both_teams_to_score,
        "is_odd": is_odd,
        "under_0_5": total_goals < 0.5,
        "under_1_5": total_goals < 1.5,
        "under_2_5": total_goals < 2.5,
        "under_3_5": total_goals < 3.5,
        "outcome": "1" if home_score > away_score else "2" if away_score > home_score else "X"
    }
    
    return result

def display_prediction(prediction):
    """
    Displays the predicted result.
    """
    print("\n=== Match Prediction ===")
    print(f"🏟️ Match: {prediction['home_team']} {prediction['home_score']} - {prediction['away_score']} {prediction['away_team']}")
    print(f"⚽ Total Goals: {prediction['total_goals']}")
    print(f"❓ Both Teams to Score: {'Yes' if prediction['both_teams_to_score'] else 'No'}")
    print(f"🧮 Total Goals Odd/Even: {'Odd' if prediction['is_odd'] else 'Even'}")
    print(f"📊 Goals Under 0.5: {'Yes' if prediction['under_0_5'] else 'No'}")
    print(f"📊 Goals Under 1.5: {'Yes' if prediction['under_1_5'] else 'No'}")
    print(f"📊 Goals Under 2.5: {'Yes' if prediction['under_2_5'] else 'No'}")
    print(f"📊 Goals Under 3.5: {'Yes' if prediction['under_3_5'] else 'No'}")
    print(f"🔮 Predicted Outcome (1=Home Win, X=Draw, 2=Away Win): {prediction['outcome']}")
    print("===")

def main():
    match_details = get_match_details()
    prediction = predict_result(match_details)
    display_prediction(prediction)

if _name_ == "_main_":
    main()
