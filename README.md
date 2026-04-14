# Abhi-fit
This is my first GitHub repository 
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        supportFragmentManager.beginTransaction()
            .replace(R.id.container, HomeFragment())
            .commit()
    }
}
class HomeFragment : Fragment(R.layout.fragment_home) {

    private lateinit var pref: PrefManager

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        pref = PrefManager(requireContext())

        val levelText = view.findViewById<TextView>(R.id.levelText)
        val streakText = view.findViewById<TextView>(R.id.streakText)
        val startBtn = view.findViewById<Button>(R.id.startWorkoutBtn)

        levelText.text = "LEVEL ${pref.getLevel()}"
        streakText.text = "🔥 Streak: ${pref.getStreak()} Days"

        startBtn.setOnClickListener {
            parentFragmentManager.beginTransaction()
                .replace(R.id.container, WorkoutFragment())
                .addToBackStack(null)
                .commit()
        }
    }
}
class WorkoutFragment : Fragment(R.layout.fragment_workout) {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {

        val completeBtn = view.findViewById<Button>(R.id.completeWorkoutBtn)
        val pref = PrefManager(requireContext())
        val levelManager = LevelManager(pref)

        completeBtn.setOnClickListener {

            levelManager.addDailyWorkoutXP()

            val calories = CaloriesManager().calculate(15)
            val totalCalories = pref.getCalories() + calories

            pref.saveCalories(totalCalories)

            Toast.makeText(context, "Workout Done +XP!", Toast.LENGTH_SHORT).show()
        }
    }
}
class LevelManager(private val pref: PrefManager) {

    fun addDailyWorkoutXP() {
        var xp = pref.getXP() + 100
        var level = pref.getLevel()

        while (xp >= level * 700) {
            xp -= level * 700
            level++
        }

        pref.saveLevelXP(level, xp)
    }
}
class PrefManager(context: Context) {

    private val prefs =
        context.getSharedPreferences("FitLevelPrefs", Context.MODE_PRIVATE)

    fun saveLevelXP(level: Int, xp: Int) {
        prefs.edit().apply {
            putInt("LEVEL", level)
            putInt("XP", xp)
            apply()
        }
    }

    fun saveCalories(cal: Int) {
        prefs.edit().putInt("CAL", cal).apply()
    }

    fun getLevel() = prefs.getInt("LEVEL", 1)
    fun getXP() = prefs.getInt("XP", 0)
    fun getStreak() = prefs.getInt("STREAK", 0)
    fun getCalories() = prefs.getInt("CAL", 0)
}
class CaloriesManager {

    fun calculate(minutes: Int): Int {
        return minutes * 6
    }
}
class NotificationHelper(private val context: Context) {

    fun showNotification() {

        val channelId = "fit_channel"

        val builder = NotificationCompat.Builder(context, channelId)
            .setSmallIcon(R.drawable.ic_launcher_foreground)
            .setContentTitle("Workout Reminder 💪")
            .setContentText("Don't skip today!")
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)

        NotificationManagerCompat.from(context).notify(1, builder.build())
    }
}
class ReminderReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        NotificationHelper(context).showNotification()
    }
}
<receiver android:name=".notifications.ReminderReceiver"/>
class ProfileFragment : Fragment(R.layout.fragment_profile) {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {

        val pref = PrefManager(requireContext())
        val caloriesText = view.findViewById<TextView>(R.id.caloriesText)

        caloriesText.text = "Calories: ${pref.getCalories()}"
    }
}