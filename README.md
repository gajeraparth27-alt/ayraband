import 'dart:async';
import 'dart:math';
import 'package:flutter/material.dart';
import 'package:fl_chart/fl_chart.dart';

void main() {
  runApp(const MyApp());
}

/* ================= APP ROOT ================= */

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData.dark(),
      home: const HomeScreen(),
    );
  }
}

/* ================= HOME WITH TABS ================= */

class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  int index = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: index == 0 ? const DashboardScreen() : const InsightsScreen(),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: index,
        onTap: (v) => setState(() => index = v),
        backgroundColor: Colors.black,
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.dashboard), label: "Dashboard"),
          BottomNavigationBarItem(icon: Icon(Icons.show_chart), label: "Insights"),
        ],
      ),
    );
  }
}

/* ================= DASHBOARD ================= */

class DashboardScreen extends StatefulWidget {
  const DashboardScreen({super.key});

  @override
  State<DashboardScreen> createState() => _DashboardScreenState();
}

class _DashboardScreenState extends State<DashboardScreen> {
  int selectedDay = 0;

  int strainScore = 6;
  int recoveryScore = 78;
  int sleepScore = 72;
  int heartRate = 68;
  int readinessScore = 80;

  String activity = "Resting";
  String stress = "Low";

  List<double> sleepTrend = [6.2, 6.8, 7.0, 7.1, 7.4];
  List<double> strainTrend = [2, 3, 4, 5, 6];
  List<double> hrTrend = [65, 67, 70, 68, 72];

  Timer? timer;
  final Random random = Random();

  @override
  void initState() {
    super.initState();
    _simulateData();
  }

  void _simulateData() {
    timer = Timer.periodic(const Duration(seconds: 3), (_) {
      setState(() {
        heartRate = 60 + random.nextInt(70);
        strainScore = min(21, strainScore + random.nextInt(2));
        recoveryScore = _nextScore(recoveryScore);
        sleepScore = _nextScore(sleepScore);

        readinessScore =
            ((recoveryScore * 0.5) + (sleepScore * 0.3) + (100 - strainScore * 3) * 0.2)
                .toInt();

        _updateActivity();
        _updateStress();

        hrTrend.removeAt(0);
        hrTrend.add(heartRate.toDouble());

        sleepTrend.removeAt(0);
        sleepTrend.add(6 + random.nextDouble() * 2.5);

        strainTrend.removeAt(0);
        strainTrend.add(strainScore.toDouble());
      });
    });
  }

  void _updateActivity() {
    if (heartRate > 120) {
      activity = "Workout";
    } else if (heartRate > 95) {
      activity = "Running";
    } else if (heartRate > 80) {
      activity = "Walking";
    } else {
      activity = "Resting";
    }
  }

  void _updateStress() {
    if (heartRate > 115 && activity == "Resting") {
      stress = "High";
    } else if (heartRate > 95) {
      stress = "Medium";
    } else {
      stress = "Low";
    }
  }

  int _nextScore(int current) {
    int change = random.nextInt(7) - 3;
    int next = current + change;
    if (next < 45) next = 45;
    if (next > 95) next = 95;
    return next;
  }

  String aiCoach() {
    if (recoveryScore > 75 && strainScore < 10) {
      return "AI Coach: Great recovery. Train hard today.";
    } else if (recoveryScore < 60) {
      return "AI Coach: Focus on recovery and light activity.";
    } else {
      return "AI Coach: Moderate training recommended.";
    }
  }

  @override
  void dispose() {
    timer?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF0B0B0B),
      body: SafeArea(
        child: SingleChildScrollView(
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              children: [
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const CircleAvatar(radius: 18),
                    Column(
                      children: [
                        const Icon(Icons.watch, size: 18),
                        Text("$readinessScore%",
                            style: const TextStyle(fontSize: 12)),
                      ],
                    )
                  ],
                ),
                const SizedBox(height: 16),
                _heartRateCard(),
                const SizedBox(height: 20),
                _statusRow(),
                const SizedBox(height: 20),

                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                  children: [
                    WhoopRing(
                      label: "Strain",
                      score: strainScore * 4,
                      color: Colors.orange,
                      onTap: () =>
                          _openDetail(context, "Strain", Colors.orange, strainScore * 4),
                    ),
                    WhoopRing(
                      label: "Recovery",
                      score: recoveryScore,
                      color: Colors.green,
                      onTap: () =>
                          _openDetail(context, "Recovery", Colors.green, recoveryScore),
                    ),
                    WhoopRing(
                      label: "Sleep",
                      score: sleepScore,
                      color: Colors.blue,
                      onTap: () =>
                          _openDetail(context, "Sleep", Colors.blue, sleepScore),
                    ),
                  ],
                ),

                const SizedBox(height: 30),

                Row(
                  children: [
                    Expanded(
                      child: MonitoringCard(
                        title: "Sleep Monitoring",
                        score: sleepScore,
                        color: Colors.blue,
                        trend: sleepTrend,
                      ),
                    ),
                    const SizedBox(width: 14),
                    Expanded(
                      child: MonitoringCard(
                        title: "Strain Monitoring",
                        score: strainScore * 4,
                        color: Colors.orange,
                        trend: strainTrend,
                      ),
                    ),
                  ],
                ),

                const SizedBox(height: 20),
                _aiCoachCard(),
              ],
            ),
          ),
        ),
      ),
    );
  }

  Widget _heartRateCard() {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.redAccent.withOpacity(0.15),
        borderRadius: BorderRadius.circular(18),
      ),
      child: Row(
        children: [
          const Icon(Icons.favorite, color: Colors.red),
          const SizedBox(width: 10),
          Text("$heartRate bpm",
              style:
                  const TextStyle(fontSize: 26, fontWeight: FontWeight.bold)),
          const Spacer(),
          SizedBox(
            width: 80,
            height: 40,
            child: LineChart(
              LineChartData(
                gridData: FlGridData(show: false),
                titlesData: FlTitlesData(show: false),
                borderData: FlBorderData(show: false),
                lineBarsData: [
                  LineChartBarData(
                    spots: List.generate(
                        hrTrend.length,
                        (i) => FlSpot(i.toDouble(), hrTrend[i])),
                    isCurved: true,
                    barWidth: 2,
                    color: Colors.red,
                    dotData: FlDotData(show: false),
                  ),
                ],
              ),
            ),
          )
        ],
      ),
    );
  }

  Widget _statusRow() {
    return Row(
      children: [
        Expanded(child: _statusCard("Activity", activity)),
        const SizedBox(width: 10),
        Expanded(child: _statusCard("Stress", stress)),
        const SizedBox(width: 10),
        Expanded(child: _statusCard("Readiness", "$readinessScore%")),
      ],
    );
  }

  Widget _statusCard(String title, String value) {
    return Container(
      padding: const EdgeInsets.all(14),
      decoration: BoxDecoration(
        color: Colors.white10,
        borderRadius: BorderRadius.circular(16),
      ),
      child: Column(
        children: [
          Text(title, style: const TextStyle(fontSize: 12)),
          const SizedBox(height: 6),
          Text(value,
              style:
                  const TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }

  Widget _aiCoachCard() {
    return Container(
      width: double.infinity,
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.greenAccent.withOpacity(0.15),
        borderRadius: BorderRadius.circular(18),
      ),
      child: Text(aiCoach(),
          style: const TextStyle(fontSize: 14, fontWeight: FontWeight.w500)),
    );
  }

  void _openDetail(
      BuildContext context, String type, Color color, int score) {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (_) =>
            DetailScreen(type: type, color: color, score: score),
      ),
    );
  }
}

/* ================= MONITOR CARD ================= */

class MonitoringCard extends StatelessWidget {
  final String title;
  final int score;
  final Color color;
  final List<double> trend;

  const MonitoringCard({
    super.key,
    required this.title,
    required this.score,
    required this.color,
    required this.trend,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white10,
        borderRadius: BorderRadius.circular(18),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(title),
          const SizedBox(height: 10),
          Text("$score%",
              style:
                  TextStyle(fontSize: 24, fontWeight: FontWeight.bold, color: color)),
          const SizedBox(height: 10),
          SizedBox(
            height: 40,
            child: LineChart(
              LineChartData(
                gridData: FlGridData(show: false),
                titlesData: FlTitlesData(show: false),
                borderData: FlBorderData(show: false),
                lineBarsData: [
                  LineChartBarData(
                    spots: List.generate(
                        trend.length,
                        (i) => FlSpot(i.toDouble(), trend[i])),
                    isCurved: true,
                    barWidth: 2,
                    color: color,
                    dotData: FlDotData(show: false),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}

/* ================= WHOOP RING ================= */

class WhoopRing extends StatelessWidget {
  final String label;
  final int score;
  final Color color;
  final VoidCallback onTap;

  const WhoopRing({
    super.key,
    required this.label,
    required this.score,
    required this.color,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: Column(
        children: [
          SizedBox(
            width: 120,
            height: 120,
            child: Stack(
              alignment: Alignment.center,
              children: [
                PieChart(
                  PieChartData(
                    startDegreeOffset: -90,
                    sectionsSpace: 0,
                    centerSpaceRadius: 40,
                    sections: [
                      PieChartSectionData(
                        value: score.toDouble(),
                        radius: 18,
                        showTitle: false,
                        color: color,
                      ),
                      PieChartSectionData(
                        value: (100 - score).toDouble(),
                        radius: 18,
                        showTitle: false,
                        color: Colors.white12,
                      ),
                    ],
                  ),
                ),
                Text("$score%"),
              ],
            ),
          ),
          const SizedBox(height: 6),
          Text(label),
        ],
      ),
    );
  }
}

/* ================= INSIGHTS SCREEN ================= */

class InsightsScreen extends StatelessWidget {
  const InsightsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF0B0B0B),
      appBar: AppBar(title: const Text("Weekly Insights")),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: const [
          InsightTile(title: "Sleep Consistency", value: "Stable"),
          InsightTile(title: "Strain Balance", value: "Moderate"),
          InsightTile(title: "Recovery Trend", value: "Improving"),
        ],
      ),
    );
  }
}

class InsightTile extends StatelessWidget {
  final String title;
  final String value;

  const InsightTile({super.key, required this.title, required this.value});

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.only(bottom: 14),
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white10,
        borderRadius: BorderRadius.circular(16),
      ),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(title),
          Text(value,
              style:
                  const TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
        ],
      ),
    );
  }
}

/* ================= DETAIL SCREEN WITH INSIGHTS ================= */

class DetailScreen extends StatelessWidget {
  final String type;
  final Color color;
  final int score;

  const DetailScreen({
    super.key,
    required this.type,
    required this.color,
    required this.score,
  });

  List<double> _trend() {
    final random = Random();
    return List.generate(7, (i) => (score - 10 + random.nextInt(20)).toDouble());
  }

  String _aiAnalysis() {
    if (score > 80) {
      return "AI Insight: Excellent condition. Your body is performing at peak.";
    } else if (score > 65) {
      return "AI Insight: Good balance but can still be optimized.";
    } else if (score > 50) {
      return "AI Insight: Moderate fatigue detected.";
    } else {
      return "AI Insight: Recovery needed.";
    }
  }

  String _recommendation() {
    if (type == "Sleep") {
      if (score > 80) return "Recommendation: Maintain your current sleep schedule.";
      if (score > 60) return "Recommendation: Improve sleep consistency.";
      return "Recommendation: Sleep earlier and reduce screen time.";
    }

    if (type == "Recovery") {
      if (score > 80) return "Recommendation: High intensity training is okay.";
      if (score > 60) return "Recommendation: Moderate training suggested.";
      return "Recommendation: Focus on recovery activities.";
    }

    if (type == "Strain") {
      if (score > 75) return "Recommendation: Reduce today's load.";
      if (score > 50) return "Recommendation: Balanced activity recommended.";
      return "Recommendation: Increase activity gradually.";
    }

    return "";
  }

  @override
  Widget build(BuildContext context) {
    final trend = _trend();

    return Scaffold(
      backgroundColor: const Color(0xFF0B0B0B),
      appBar: AppBar(title: Text("$type Insights")),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          Center(
            child: Text(
              "$score%",
              style: TextStyle(
                  fontSize: 44, fontWeight: FontWeight.bold, color: color),
            ),
          ),
          const SizedBox(height: 20),

          Container(
            height: 200,
            padding: const EdgeInsets.all(12),
            decoration: BoxDecoration(
              color: Colors.white10,
              borderRadius: BorderRadius.circular(16),
            ),
            child: LineChart(
              LineChartData(
                gridData: FlGridData(show: false),
                titlesData: FlTitlesData(show: false),
                borderData: FlBorderData(show: false),
                lineBarsData: [
                  LineChartBarData(
                    spots: List.generate(
                        trend.length, (i) => FlSpot(i.toDouble(), trend[i])),
                    isCurved: true,
                    barWidth: 3,
                    color: color,
                    dotData: FlDotData(show: false),
                    belowBarData: BarAreaData(
                      show: true,
                      color: color.withOpacity(0.2),
                    ),
                  ),
                ],
              ),
            ),
          ),

          const SizedBox(height: 20),

          _metricCard("Trend Average", "${trend.reduce((a, b) => a + b) ~/ trend.length}%"),
          _metricCard("Body Status", _statusText(score)),
          _metricCard("AI Analysis", _aiAnalysis()),
          _metricCard("AI Recommendation", _recommendation()),
        ],
      ),
    );
  }

  String _statusText(int score) {
    if (score > 80) return "Peak";
    if (score > 65) return "Good";
    if (score > 50) return "Moderate";
    return "Low";
  }

  Widget _metricCard(String title, String value) {
    return Container(
      margin: const EdgeInsets.only(bottom: 14),
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white10,
        borderRadius: BorderRadius.circular(16),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(title, style: const TextStyle(color: Colors.white60)),
          const SizedBox(height: 6),
          Text(value,
              style:
                  const TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }
}
