from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///job_recommendation.db"
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    skills = db.Column(db.String(200), nullable=False)
    experience_level = db.Column(db.String(100), nullable=False)
    preferences = db.Column(db.String(200), nullable=False)

class Job(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    job_title = db.Column(db.String(100), nullable=False)
    company = db.Column(db.String(100), nullable=False)
    required_skills = db.Column(db.String(200), nullable=False)
    location = db.Column(db.String(100), nullable=False)
    job_type = db.Column(db.String(100), nullable=False)
    experience_level = db.Column(db.String(100), nullable=False)

@app.route("/users", methods=["POST"])
def create_user():
    data = request.get_json()
    user = User(name=data["name"], skills=data["skills"], experience_level=data["experience_level"], preferences=data["preferences"])
    db.session.add(user)
    db.session.commit()
    return jsonify({"message": "User created successfully"}), 201

@app.route("/jobs", methods=["GET"])
def get_jobs():
    user_id = request.args.get("user_id")
    user = User.query.get(user_id)
    if user:
        jobs = Job.query.filter(Job.required_skills.contains(user.skills)).filter(Job.experience_level == user.experience_level).filter(Job.location.in_(user.preferences["locations"])).filter(Job.job_type == user.preferences["job_type"]).all()
        return jsonify([job.to_dict() for job in jobs])
    return jsonify({"message": "User not found"}), 404

if __name__ == "__main__":
    app.run(debug=True)
