from flask import Flask, render_template, request, redirect, url_for, jsonify
import json
import os
from datetime import datetime, timedelta

app = Flask(__name__)
data_file = 'data/limits.json'

def load_limits():
    if os.path.exists(data_file):
        with open(data_file, 'r') as file:
            return json.load(file)
    return {}

def save_limits(limits):
    with open(data_file, 'w') as file:
        json.dump(limits, file)

@app.route('/')
def index():
    limits = load_limits()
    return render_template('index.html', limits=limits)

@app.route('/set_limit', methods=['POST'])
def set_limit():
    app_name = request.form['app_name']
    time_limit = int(request.form['time_limit'])
    end_time = datetime.now() + timedelta(minutes=time_limit)
    
    limits = load_limits()
    limits[app_name] = end_time.isoformat()
    save_limits(limits)
    
    return redirect(url_for('index'))

@app.route('/check_limits')
def check_limits():
    limits = load_limits()
    alerts = []
    now = datetime.now()

    for app_name, end_time in limits.items():
        end_time = datetime.fromisoformat(end_time)
        if now >= end_time:
            alerts.append(app_name)

    return jsonify(alerts)

if __name__ == '__main__':
    app.run(debug=True)

