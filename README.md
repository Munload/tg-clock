package com.munload.vpn

import android.app.Activity
import android.content.Intent
import android.net.VpnService
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.compose.setContent
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Settings
import androidx.compose.material.icons.filled.KeyboardArrowRight
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                Surface(modifier = Modifier.fillMaxSize(), color = Color(0xFFF7F9FC)) {
                    VpnScreen()
                }
            }
        }
    }
}

@Composable
fun VpnScreen() {
    val context = LocalContext.current
    // Состояние: подключен VPN или нет (для смены цвета и текста)
    var isConnected by remember { mutableStateOf(false) }

    val launcher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == Activity.RESULT_OK) {
            startMyVpnService(context)
            isConnected = true
        }
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        // --- ВЕРХНЯЯ ПАНЕЛЬ ---
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text(
                text = "Munload VPN",
                fontSize = 24.sp,
                fontWeight = FontWeight.Bold,
                color = Color.Black
            )
            Icon(
                imageVector = Icons.Default.Settings,
                contentDescription = "Settings",
                tint = Color.Gray,
                modifier = Modifier.size(28.dp)
            )
        }

        Spacer(modifier = Modifier.height(40.dp))

        // --- КНОПКА ВЫБОРА ЛОКАЦИИ ---
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .clip(RoundedCornerShape(16.dp))
                .background(Color.White)
                .clickable { /* Здесь потом будет выбор сервера */ }
                .padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text(text = "🇩🇪", fontSize = 24.sp)
            Spacer(modifier = Modifier.width(12.dp))
            Text(
                text = "Frankfurt-bYyU",
                fontSize = 16.sp,
                fontWeight = FontWeight.Medium,
                modifier = Modifier.weight(1f)
            )
            Icon(
                imageVector = Icons.Default.KeyboardArrowRight,
                contentDescription = "Select",
                tint = Color.Gray
            )
        }

        Spacer(modifier = Modifier.weight(1f)) // Отталкиваем кнопку в центр

        // --- БОЛЬШАЯ КНОПКА ВКЛЮЧЕНИЯ ---
        val mainColor = if (isConnected) Color(0xFF34C759) else Color(0xFFE0E0E0)
        val iconColor = if (isConnected) Color.White else Color.Gray

        Box(
            modifier = Modifier
                .size(220.dp)
                .clip(CircleShape)
                .background(mainColor.copy(alpha = 0.2f)) // Светлый фон вокруг
                .clickable {
                    if (!isConnected) {
                        val intent = VpnService.prepare(context)
                        if (intent != null) {
                            launcher.launch(intent)
                        } else {
                            startMyVpnService(context)
                            isConnected = true
                        }
                    } else {
                        // Логика отключения (пока просто меняем состояние)
                        stopMyVpnService(context)
                        isConnected = false
                    }
                },
            contentAlignment = Alignment.Center
        ) {
            Box(
                modifier = Modifier
                    .size(160.dp)
                    .clip(CircleShape)
                    .background(mainColor),
                contentAlignment = Alignment.Center
            ) {
                // Вместо молнии используем встроенную иконку питания
                Icon(
                    imageVector = Icons.Default.PowerSettingsNew, // Потом заменим на молнию
                    contentDescription = "Power",
                    tint = iconColor,
                    modifier = Modifier.size(64.dp)
                )
            }
        }

        Spacer(modifier = Modifier.height(24.dp))

        // --- ТЕКСТ СТАТУСА ---
        Text(
            text = if (isConnected) "Подключено" else "Не подключено",
            fontSize = 18.sp,
            color = if (isConnected) Color(0xFF34C759) else Color.Gray
        )

        Spacer(modifier = Modifier.weight(1f)) // Отталкиваем вниз
    }
}

// Вспомогательные функции для запуска/остановки
fun startMyVpnService(context: android.content.Context) {
    val intent = Intent(context, MyVpnService::class.java).apply { action = "START" }
    context.startService(intent)
}

fun stopMyVpnService(context: android.content.Context) {
    val intent = Intent(context, MyVpnService::class.java).apply { action = "STOP" }
    context.startService(intent)
}
